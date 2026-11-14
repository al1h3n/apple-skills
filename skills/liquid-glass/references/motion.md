# Motion: springs, interaction choreography, frame pacing

Liquid Glass "behaves and moves organically in a manner that feels more like a lightweight
liquid" [Apple]. The look is half of it; the other half is motion that is **spring-driven,
interruptible, velocity-preserving and frame-rate independent**. Most "cheap copies" feel
wrong because they integrate forces with Euler steps, use collisions, or fade alpha.

---

## 1. The spring model Apple uses

Apple parameterises springs by **duration** (perceptual settle time) and **bounce**
(WWDC23 "Animate with springs"; SwiftUI `spring(duration:bounce:)`):

```
mass      = 1
ω0        = 2π / duration                  (natural frequency)
stiffness = ω0²
ζ         = 1 − bounce          if bounce ≥ 0   (damping ratio)
ζ         = 1 / (1 + bounce)    if bounce < 0   (over-damped, "flatter")
damping   = 2 ζ ω0
```

SwiftUI presets (use the same numbers everywhere):

| Preset | duration | bounce | Use |
|---|---|---|---|
| `.smooth` | 0.5 | 0 | appearance changes, materialize, colour/tint |
| `.snappy` | 0.5 | 0.15 | layout / morph / frame changes |
| `.bouncy` | 0.5 | 0.3 | release after press, menu pop-open |
| `.interactiveSpring` | 0.15 (response) | ≈0.14 (dampingFraction 0.86) | follow the finger |

### Solve it analytically, never with Euler

With `x = value − target`, `v = velocity`, after time `t`:

```
ζ < 1  (under-damped):  ωd = ω0·√(1−ζ²)
    x(t) = e^(−ζω0t) · ( x0·cos ωd t + (v0 + ζω0x0)/ωd · sin ωd t )
ζ = 1  (critical):
    x(t) = e^(−ω0t) · ( x0 + (v0 + ω0x0)·t )
ζ > 1  (over-damped):   r1,2 = −ω0(ζ ∓ √(ζ²−1)),  c2 = (v0 − r1x0)/(r2 − r1),  c1 = x0 − c2
    x(t) = c1·e^(r1t) + c2·e^(r2t)
v(t) = dx/dt (differentiate the same expression)
```

Properties that make it feel native:
- **Exact for any `dt`** — 60, 120 Hz or a dropped frame produce the same curve.
- **Retargeting** (`target = new`) keeps the current value and velocity, so interrupting a
  morph mid-flight continues smoothly — this is SwiftUI's behaviour.
- Rest when `|x| < ε` and `|v| < ε` (ε ≈ 0.001 pt/scale units) → snap and stop scheduling
  frames. Past ~1024 an `f32` can't land within ε of the target (steps are 1/8192 at 1024),
  so also rest when the remaining swing `hypot(x, v/ω0)` fits in ~16 float steps of the
  target, or when a step rounds back to the same value; otherwise the host renders forever.
- Clamp `bounce` to ±0.95 before computing ζ: at 1 the spring rings forever and at −1 ζ is
  infinite. Land at once on a NaN or infinite target, step or spring.

Rust (as in Liquid Rust `motion.rs`):

```rust
/// Advances displacement `x0` and velocity `v0` by `t` seconds. Returns (x, v).
fn solve(w0: f32, zeta: f32, x0: f32, v0: f32, t: f32) -> (f32, f32) {
    if zeta < 1.0 {
        let a = zeta * w0;
        let wd = w0 * (1.0 - zeta * zeta).sqrt();
        let (s, c) = (wd * t).sin_cos();
        let e = (-a * t).exp();
        let b = (v0 + a * x0) / wd;
        (e * (x0 * c + b * s), e * (v0 * c - (x0 * wd + a * b) * s))
    } else if zeta == 1.0 {
        let e = (-w0 * t).exp();
        let b = v0 + w0 * x0;
        (e * (x0 + b * t), e * (v0 - w0 * b * t))
    } else {
        let q = (zeta * zeta - 1.0).sqrt();
        let (r1, r2) = (-w0 * (zeta - q), -w0 * (zeta + q));
        let c2 = (v0 - r1 * x0) / (r2 - r1);
        let c1 = x0 - c2;
        let (e1, e2) = ((r1 * t).exp(), (r2 * t).exp());
        (c1 * e1 + c2 * e2, c1 * r1 * e1 + c2 * r2 * e2)
    }
}
```
Under-damped velocity simplifies because `b·ωd − a·x0 = v0`. Copy this (or Liquid Rust's
`motion.rs`) instead of re-deriving — sign errors in the derivative are the classic bug.

TypeScript/web: identical maths; drive from `requestAnimationFrame` timestamps.

## 2. Measured on iOS 27, frame by frame

A 60 fps recording of an iPhone on iOS 27 (15,650 frames, 260 s) was stepped frame by frame:
Control Center opening, its modules expanding and collapsing, its pages swiping, edit mode
(dragging a glass pill over other controls), lock-screen customization (sheets, the font and
colour sliders, a segmented Glass/Solid control), the camera (mode switch, zoom dial, record
button), Photos (toolbar morphs, the share sheet growing out of its button and closing back).
All 119 motion events were laid out frame by frame; surfaces were tracked edge by edge.
The display runs at 120 Hz: anything fast shows twice in one 60 fps frame (doubled text is
motion, not a rendering bug).

### 2.1 Surfaces launch; they never ease in

A module expanding (top edge tracked, progress = share of the way):

| time | 17 ms | 33 ms | 50 ms | 100 ms | 150 ms | 200 ms | ≈350 ms |
|---|---|---|---|---|---|---|---|
| progress | 0.20 | 0.35 | 0.50 | 0.77 | 0.91 | 0.98 | settled |

That is a first-order exponential, `p(t) = 1 − e^(−t/τ)` with **τ ≈ 72 ms**: full speed on the
very first frame, then a soft landing, no overshoot. Sheets and the share sheet follow the
same shape. **A spring from rest does not fit**: `spring(0.45, 0)` from rest is at 0.16 after
50 ms (measured 0.50); its ease-in is what makes web copies feel laggy and makes a growing
menu look squeezed, small for too long and then smashed open.

The launched spring reproduces it exactly. A critically damped spring (bounce 0) whose start
velocity is `ω0 × distance` toward the target has `v0 + ω0·x0 = 0`, so

```
x(t) = x0 · e^(−ω0·t)          (x = value − target, ω0 = 2π / duration)
```

| Motion | Spring (launch 1) | τ | 95 % at |
|---|---|---|---|
| **Emerge** (a surface opening: menu, popover, sheet, expanded module) | `0.45 s, 0` | 72 ms | 0.21 s |
| **Recede** (closing back into its button) | `0.3 s, 0` | 48 ms | 0.14 s |
| **Knob settle** (a lens dropping back to a solid knob) | `0.18 s, 0` | 29 ms | 0.09 s |

```ts
// Launch: retarget with velocity toward the target (keep a faster flick already under way).
launch(target, { duration, bounce }, launch = 1) {
  const speed = (target - this.value) * (2 * Math.PI / duration) * launch;
  if (!(this.velocity * speed > 0 && Math.abs(this.velocity) > Math.abs(speed))) this.velocity = speed;
  this.target = target; this.timing = { duration, bounce };
}
```

Liquid Rust 0.2.6: `Animated::launch_to`, `Spring::EMERGE` / `RECEDE` / `KNOB_SETTLE`,
`Scene::set_frame_launched`. CSS: sample the curve into `linear()` (BitsUILiquidGlass
`springEasing`):

```css
--emerge: linear(0, 0.2524, 0.4411, 0.6058, 0.7053, 0.7797, 0.8353, 0.8769, 0.9131, 0.9351, 0.9515, 0.9637, 0.9744, 0.9809, 0.9857, 0.9893, 0.992, 0.9944, 0.9958, 0.9968, 0.9976, 0.9982, 0.9988, 0.9991, 1); /* 520 ms */
--recede: linear(0, 0.4076, 0.6171, 0.7732, 0.8534, 0.9131, 0.9439, 0.9667, 0.9785, 0.9873, 0.9918, 0.9951, 0.9968, 0.9981, 0.9988, 0.9993, 1); /* 360 ms */
--knob-settle: linear(0, 0.5167, 0.7665, 0.8695, 0.9369, 0.9695, 0.9853, 0.9918, 0.996, 0.9981, 0.9991, 0.9995, 1); /* 240 ms */
```

### 2.2 What else the frames show

- **Menu from a button = the button becomes the menu.** The share sheet starts as the share
  button (its frame and corners), grows as *one* shape and lands; the header text appears
  small and soft near the button and sharpens as the surface lands. No liquid neck stretches
  between button and menu and the button never bulges. Closing reverses it on the recede curve
  and the shape is gone the instant it reaches the button. (In Liquid Rust: `morph_from` /
  `shrink_into`, never `expand_from` with a shared container for this.)
- **Content arrives soft.** Content inside a surface that is still moving is blurred and
  sharpens over roughly the first half of the motion (≈8 pt → 0); it is never revealed at full
  sharpness by a clip.
- **Closing is quicker than opening** (about 7 frames for most of the travel versus ~20).
- **Knob lens.** Held, a slider knob is a clear lens about 1.5× its size: the track shows
  through it, frosted, bent at its rim, with a lit edge. Released, it shrinks back to the white
  knob in 5–6 frames (≈0.1 s), whitening as it shrinks. **No wobble, no jelly.**
- **Dragged glass keeps its shape.** A glass pill dragged across Control Center in edit mode
  lifts ≈1.08×, turns translucent over the controls below, and does not stretch even in a
  fast flick; dropped, it flies to its slot on the emerge curve. Keep stretch ≤ 10 % with a
  bounce of at most 0.1, or skip it.
- **Pages and lists stagger.** Swiping Control Center pages, rows arrive top first, each
  scaling up a little (≈0.92 → 1) as it lands; the outgoing page leaves on the same curve.
- **Apple's own glitch, to avoid.** Leaving lock-screen customization, the clock first jumps
  to its small size and then animates a second time. Never run two animations in sequence on
  one property, and never jump before animating: interpolate once, from the value on screen,
  keeping velocity.

### 2.3 The switch, iOS 26.5 vs 27 (every frame of a side-by-side recording)

One tap on iOS 27, about 25 frames (0.42 s):

| Frames | What happens |
|---|---|
| 1–2 | The white knob dissolves into frost; it does not grow yet |
| 3–8 | It swells into a lens ~1.45× wide and ~1.3× tall, overhanging the track at both ends. Inside, the track is magnified and bent; thick white crescents sit on the top and bottom inner rims; the outer rim is a dark grey hairline |
| 9–14 | The lens slides across while big; the new colour (green) shows inside the lens before the track turns |
| 15–23 | It shrinks as it lands and the white fills back in from the side |

Held, the lens stays up. A quick click still plays the whole sequence. iOS 26.5 is the same
choreography with a thin rainbow (dispersion) rim and a lighter edge; 27 is darker and calmer.
CSS: `liquid-glass.css` `.lg-switch` with `data-flipping` for 300 ms on click; WebGPU: three
`set_frame` calls (lens at the start, lens at the end after 60 ms, knob after 250 ms), bounce 0.

### 2.4 Siri (iOS 27, every frame)

The island swells down into a round drop (~0.5 s) whose lower rim bends what is under it, then
stretches sideways into the pill (~0.33 s) while the status bar blurs away under it. Words never
slide: the old text blurs and fades in ~4 frames, the new sharpens in place over ~12. The
material is black at the top fading into clear glass at the bottom edge.

### 2.5 More from the SiriAI recordings and trailer (every frame)

- **The working ring.** While Siri works, a ring of 7 glowing spheres sits left of the status
  text. It is a ring in 3D: it turns about its centre while its tilt swings and precesses, so
  spheres swell and brighten coming forward, shrink and dim going back, and pass behind each
  other (a flat spinner reads cheap). Each sphere has a soft bloom; colour bluish white on the
  black island. Between "Searching" and "Looking into it" the ring tumbles further (spheres
  cluster and spread), it never jumps.
- **Status text.** Grey at about 55 % white, cross-fading through a blur (old out in ~4 frames,
  new in over ~12); a slow light passes through it. The answer text replaces the ring at the
  same left edge, white and larger.
- **Rim light.** Once the answer has landed, a spectral band (blue → white → orange) runs once
  along the bottom rim, left to right (~1 s), with a soft glow rising from it.
- **Answer card.** The island grows down into the card in ~15 frames. Its text sharpens from a
  blur line by line, top first, the last line a beat later (~70 ms apart). An embedded card
  (weather, map) first shows as a coloured glow line at the card's bottom rim, then slides up
  into place. Collapsing back takes ~9 frames, the content scaling and blurring with the shape.
- **Menus are blobs first.** Photos' "Filter by" menu starts as a small blurred *round* blob at
  its button, rounder than the menu, and its corners tighten to the menu's own as it lands
  (~12 frames). CSS: animate `border-radius` from a capsule as part of the emerge keyframes.
- **Notifications rise.** A lock-screen notification appears at ≈ 0.5× near the bottom,
  translucent and soft, grows to full size as it rises, then slides up into the stack.
- **App switcher.** The home screen scales up a little and blurs within 4 frames; the app cards
  enter launched from the side.
- **Emoji and icons.** The trailer shows emoji as Apple's 3D glyphs, large and alone on black
  (never flat line art), and app icons as glossy glass objects (a lit sphere over the symbol).

## 3. Interaction choreography

Values marked [Apple] are described in WWDC25; numbers are [Derived] from frame-stepping
iOS 26/27 recordings and tuned in Liquid Rust — tune by eye, keep the *shape*.

| Moment | What moves | Spring |
|---|---|---|
| **Press down** (`.interactive`) | scale 1 → 1 + min(0.15, 10 pt / size); glow intensity 0 → 1 starting at the touch point; glow radius 0 → 1.2× element size; light "energizes" (specular ×1.6) [Apple: flexes and energizes with light] | scale `0.25 s, 0`; glow `0.2 s, 0`; radius `0.45 s, 0` |
| Glow spill | the glow is a scene light, so nearby glass in the same region lights up too [Apple] | — |
| **Release** (a glass button) | scale → 1, glow → 0 | scale `0.5 s, 0.35` (one visible overshoot); glow `0.6 s, 0` |
| **Release** (a knob's lens) | lens → solid knob, ≈1.5× → 1× | `0.18 s, 0` launched (§2.2): no wobble |
| **Pointer hover** (macOS/iPad) | scale 1 → 1.03, specular ×1.2 | `0.3 s, 0` |
| **Drag a lens / knob** | position follows finger; a slight **stretch along velocity** (≤ 10 %, area-preserving), lens lifted: scale 1 → ~1.3, refraction stronger (magnifier over the value) [Apple: "elements can lift up into Liquid Glass temporarily"] | follow `0.15 s, 0`; stretch `0.3 s, 0.1` (no jelly: iOS 27 shows none, §2.2); lift `0.3 s, 0.15` |
| **Morph / layout** (glassEffectID) | frame x, y, w, h and corner radius as independent springs; neighbours fuse/split through the SDF union automatically | `0.5 s, 0.15` |
| **Menu / popover / sheet from button** | the button *becomes* the menu: one shape launched from the button's frame and corners, no neck; thickness (frost, shadow, face opacity) ramps up with size; content sharpens from a blur [Apple: "bubble simply pops open"] | frame `0.45 s, 0` launched (emerge); close `0.3 s, 0` launched (recede); content blur 8 → 0 pt over the first ≈55 % |
| **Materialize in/out** | lensing (refraction, frost, rim) 0 → 1, plus ≤4 % scale; **never** alpha | in `0.4 s, 0`; out `0.3 s, 0` |
| **Lock/unlock, focus change** | light angle sweeps around the silhouette [Apple] | `0.8 s, 0` |
| **Device tilt** | light angle follows attitude (gyro / pointer) | `0.3 s, 0` follow |
| **Light↔dark flip** of small glass | face tone crossfades | `0.35 s, 0` |

Stretch maths (area-preserving squash & stretch along drag direction `d̂`):

```
s       = clamp(|v| / 2500 pt·s⁻¹, 0, 1) · 0.25        // target elongation
scale ∥ = 1 + s_spring
scale ⊥ = 1 / (1 + s_spring)
SDF: rotate p into (d̂, d̂⊥), divide components by the two scales, evaluate, multiply the
distance by min(scale ∥, scale ⊥) to keep it ≈ Euclidean.
```

Merging and splitting **are not physics**. Do not add collision response between blobs.
Each shape springs toward its own target; the smooth union (`k` = container spacing) makes
the liquid neck form and snap. That is why Apple's version never jitters.

## 4. Frame pacing

- Drive from the display: `requestAnimationFrame` (web), `CADisplayLink` (iOS),
  `winit` `RedrawRequested` + `request_redraw()` (native Rust). Use the frame timestamp
  delta, clamp to ≤ 1/30 s to avoid explosions after a stall (analytic springs are stable
  anyway, the clamp is for visual continuity).
- **Render on demand**: when every spring is at rest and the backdrop hasn't changed, stop
  requesting frames. Liquid Rust's `Scene::update(dt) -> bool` returns whether anything is
  still animating.
- Present with `PresentMode::Fifo` (vsync) or `Mailbox`; on ProMotion/120 Hz screens the
  analytic springs are automatically smooth. iOS runs these motions at 120 Hz: a 60 fps
  recording shows two exposures of anything fast, so judge timing from edges, not ghosts.
- Keep CPU work per frame O(shapes) and allocation-free; upload one small storage buffer.

## 5. Reduce Motion

- Every spring: bounce → 0 (critically damped), duration ×0.8.
- No stretch, no press scale, glow intensity ×0.5, light sweep disabled.
- Morphs still happen (they convey structure) but without overshoot.
