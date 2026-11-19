# Motion: springs, measured curves, choreography

Everything = spring. Interruptible, velocity kept, exact at any frame rate. Cheap copies fail on: Euler steps, ease-in on finger-sent motion, alpha fades, collision physics. Why: `principles.md`. Numbers = current OS (27); older in `versions/26.md`.

## 1. Spring model

Apple: `spring(duration:bounce:)` [Apple, WWDC23 "Animate with springs"].

```
mass      = 1
ω0        = 2π / duration
stiffness = ω0²
ζ         = 1 − bounce          bounce ≥ 0
ζ         = 1 / (1 + bounce)    bounce < 0   (over-damped)
damping   = 2 ζ ω0
```

| Preset | duration | bounce | Use |
|---|---|---|---|
| smooth | 0.5 | 0 | appearance, tint, tone |
| snappy | 0.5 | 0.15 | layout, frame changes from rest |
| bouncy | 0.5 | 0.3 | rare: playful arrival |
| interactive | 0.15 | ≈ 0.14 | follow the finger |

### Solve analytically. Never Euler

`x = value − target`, `v = velocity`, after `t`:

```
ζ < 1:  ωd = ω0·√(1−ζ²)
        x(t) = e^(−ζω0t) · ( x0·cos ωd t + (v0 + ζω0x0)/ωd · sin ωd t )
ζ = 1:  x(t) = e^(−ω0t) · ( x0 + (v0 + ω0x0)·t )
ζ > 1:  r1,2 = −ω0(ζ ∓ √(ζ²−1)),  c2 = (v0 − r1x0)/(r2 − r1),  c1 = x0 − c2
        x(t) = c1·e^(r1t) + c2·e^(r2t)
v(t) = dx/dt of the same expression
```

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

Under-damped velocity simplifies because `b·ωd − a·x0 = v0`. Copy; do not re-derive (sign errors in the derivative = classic bug).

Rules:
- **Exact for any `dt`**: 60 Hz, 120 Hz, dropped frame → same curve.
- **Retarget keeps value + velocity.** Interrupting mid-flight continues smoothly.
- **Rest**: `|x| < ε` and `|v| < ε` (ε ≈ 0.001) → snap, stop requesting frames. Above ~1024 an `f32` cannot land within ε (step 1/8192 at 1024): also rest when remaining swing `hypot(x, v/ω0)` fits in ~16 float steps, or when a step rounds to the same value. Else the host renders forever.
- Clamp `bounce` to ±0.95 (1 rings forever, −1 → ζ infinite). NaN/∞ target → land at once.

## 2. Launched motion

Anything a finger sent (tap result, flick, dismissal) **starts at full speed on frame 1** and lands exponentially. No ease-in.

Measured: progress 0.20 at 17 ms, 0.50 at 50 ms, 0.77 at 100 ms, 0.91 at 150 ms, 0.98 at 200 ms = `1 − e^(−t/τ)` [M8]. A spring from rest does not fit: `spring(0.45, 0)` from rest is at 0.16 after 50 ms (measured 0.50). That ease-in is why web copies feel late and growing menus look squeezed.

Critically damped spring with start velocity `ω0 × distance` toward target → `v0 + ω0·x0 = 0`:

```
x(t) = x0 · e^(−ω0·t)          τ = duration / 2π
```

```ts
// Launch: retarget with velocity toward the target. Keep a faster flick already under way.
launch(target, { duration, bounce }, launch = 1) {
  const speed = (target - this.value) * (2 * Math.PI / duration) * launch;
  if (!(this.velocity * speed > 0 && Math.abs(this.velocity) > Math.abs(speed))) this.velocity = speed;
  this.target = target; this.timing = { duration, bounce };
}
```

| Motion | Spring, launched | τ | 95 % at | Source |
|---|---|---|---|---|
| **Emerge**: surface opens (menu, popover, sheet) | 0.45 s, 0 | 72 ms | 0.21 s | [M8] sheets τ 65, [M6] τ 75 |
| **Recede**: closes into its source | 0.30 s, 0 | 48 ms | 0.14 s | [M8] close 0.2 s, [M7] 0.17–0.2 s |
| Dismissal of a banner | 0.47 s, 0 | 75 ms | 0.22 s | [M4] |
| Summon (island → drop) | 0.38 s, 0 | 60 ms | 0.18 s | [M6] |
| Knob settle (lens → knob) | 0.18 s, 0 | 29 ms | 0.09 s | [M5][M8] |
| Foreground layer returns | 0.69 s, 0 | 110 ms | 0.33 s | [M1] |
| Page / sheet after a flick | 0.45 s, **0.1**, velocity = pointer's | 60–67 ms | 0.2 s, 1–2 % overshoot | [M8] |

CSS: sample the curve into `linear()`; generate from the same constants as the engine, never hand-type (duration of the CSS transition = settle time, not spring `duration`):

```css
--emerge: linear(0, 0.2524, 0.4411, 0.6058, 0.7053, 0.7797, 0.8353, 0.8769, 0.9131, 0.9351, 0.9515, 0.9637, 0.9744, 0.9809, 0.9857, 0.9893, 0.992, 0.9944, 0.9958, 0.9968, 0.9976, 0.9982, 0.9988, 0.9991, 1); /* 520 ms */
--recede: linear(0, 0.4076, 0.6171, 0.7732, 0.8534, 0.9131, 0.9439, 0.9667, 0.9785, 0.9873, 0.9918, 0.9951, 0.9968, 0.9981, 0.9988, 0.9993, 1); /* 360 ms */
```

### From rest (not launched)

Arrivals nobody sent, carrying something to read. Only place with bounce:

| Motion | Spring from rest | Overshoot | Source |
|---|---|---|---|
| Banner arrival, after a 0.1 s anchor drop | 0.42 s, 0.2 | 1.2 % | [M4] |
| Assistant pill → full-width bar | 0.5 s, 0.15 | ≈ 2 % | [M6] |
| Lock Screen notification rise, 0.25 → 1 | ≈ 0.23 s, 0 | none, then 6 pt drift | [M7] |
| Wallpaper zoom-out 1.10 → 1 | 1.0 s, 0 | none | [M1] |

## 3. Choreography

| Moment | What moves | Timing | Source |
|---|---|---|---|
| **Press down** | scale +5 pt, cap 15 % (44 pt button → ×1.08–1.15). Lights from the finger, never to pure white; glyph stays readable. **Neighbours do not fuse** | peak in 35–65 ms | [M3][M7][M8] |
| **Release** (button) | scale back, no overshoot seen; light outlasts shape | shape 70–100 ms; glow-out τ 90 ms | [M3] |
| **Hover** (pointer) | scale ×1.03, highlight ×1.2 | 0.3 s, 0 | [D] |
| **Knob press** | white knob → frost (1–2 frames, no growth) → swells to lens ×1.6 | 33–130 ms | [M5] |
| **Knob travel** | lens slides at full size; new colour shows **inside lens first**, track outside changes as lens passes | 130–230 ms | [M5] |
| **Knob release** | shrinks, whitens from leading side. No wobble | whole tap ≈ 0.38 s. Held: lens stays up | [M5] |
| **Drag** | lift ×1.08–1.10, face thinner (what is under tints it). **No stretch, no wobble** | follow 0.15 s | [M8] |
| **Bar changes mode** | each button morphs old frame → new frame, straight. Two that become one merge once. New button slides from under neighbour. Title slot fixed | 0.15–0.18 s to sharp; glyphs out 50 ms, in 70 ms | [M3] |
| **Surface from a control** | control → small blob → dome (round, no corners) → corners tighten at ≈ 60 % of travel. Content inside from frame 1, soft, **bent at the growing edge**. Control absent while open | 0.27 s open (emerge); 0.17–0.2 s close (recede) | [M7][M8] |
| **Content in a moving surface** | soft (≈ 8 pt blur) → sharp on landing. Never revealed sharp by a clip | first ≈ 55 % of travel | [M8] |
| **Materialize in place** (no source) | scale 0.5 → 1, blur → 0. No slide | 0.13 s, solid at 0.25 s | [M8] |
| **Dematerialize in place** | face thins first, rim last (outline ghosts 2–3 frames). No slide, no scale | 0.18 s | [M8] |
| **Cross-fade in place** (toolbar ↔ tools) | old blurs out, new sharpens | 0.13 s; secondary items in steps of 0.1 s | [M8] |
| **Text swap** | old: blur + dim + 2 pt up, 65 ms. Gap 33 ms. New: 2 pt low + blurred → sharp, 120 ms. **Never sideways** | ≈ 0.2 s | [M6] |
| **Lines reveal** | each sharpens from blur, top first | 50 ms per line | [M7] |
| **Rows on a page** | own spring each → lag ≈ 1 frame per row emerges. No timer | none | [M8] |
| **Reflow** (edit mode) | displaced neighbour waits, then moves fast | dwell 0.6 s, move ≤ 80 ms | [M8] |
| **Shape morph** (circle ↔ rounded square) | one shape, corners morph | 0.12 s | [M8] |
| **Colour change** (shutter white → red) | interpolate colour. Never through transparent | 0.17 s | [M8] |
| **Value under finger** | updates in place. No rolling digits | instant | [M8] |
| **Accent on selection** | applied after the selection settles | +0.2 s | [M8] |
| **Light/dark flip** | `design-rules.md` §5 | → light 0.25 s, → dark 0.1 s dwell + 0.4 s | [M8] |
| **Backdrop on return** | `effects.md` §1 | none | [M1] |

Merge and split **are not physics**. Each shape springs to its own target; smooth union (`k` = container spacing) forms and breaks the neck. No collision response → no jitter.

Never two animations in sequence on one property (Apple's own clock-resize stumble [M8]): interpolate once, from the value on screen, velocity kept.

### macOS 27 [M9]

| Motion | Measured |
|---|---|
| Dock magnification | layout, not scale: full size in ≈ 0.1 s, ≈ 8 % overshoot, settled by 0.2 s; moving along the Dock follows the pointer directly (a spring with response 0.1 s, damping 0.63) |
| Genie (minimize, restore) | ≈ 0.5 s; a thin strip curving out of the icon, a funnel pinned at the icon with one side almost straight, no overshoot; draw it as strips mapped onto exact quadrilaterals (projective), so the outline is smooth |
| Held equalizer point | a dome lens ≈ 4.5× the dot, no flat top, minified middle, mirrored rim, faint blue fringe; follows the pointer, shrinks back on release |
| Listening Mode slider | the selection becomes a wide knob (≈ 2:1) that stretches with speed, smears along its path when flung (motion blur about one frame) and follows the pointer past the track's end; let go there, it morphs into the option under the pointer in ≈ 8 frames |
| Tapped segment or tab | the selection slides there with a little give (≈ 0.4 s, bounce 0.12); a launched 0.14 s curve reads as a jump |

## 4. Closing is faster than opening

Everywhere: sheet 0.27 s open / 0.2 s close; banner 0.42 s / τ 75 ms; home layer leaves in 0.1 s, returns in 0.6 s [M1][M4][M8]. Person already decided → get out of the way. No bounce on anything leaving.

## 5. Frame pacing

- Drive from display: `requestAnimationFrame`, `CADisplayLink`, `RedrawRequested`. Use timestamp delta; clamp ≤ 1/30 s after a stall (maths stable anyway; clamp = visual continuity).
- **Render on demand**: all springs at rest + backdrop unchanged → stop requesting frames. Idle = 0 fps.
- 120 Hz devices: analytic springs need nothing extra. Reading recordings: a 60 fps capture of 120 Hz shows two exposures of anything fast → judge timing from edges, not ghosts.
- Per frame: O(shapes) CPU, no allocation, one small buffer upload.

## 6. Reduce Motion

- Bounce → 0. Duration ×0.8.
- No press scale, no travel, no backdrop zoom, no anchor drop, no light sweep. Glow ×0.5.
- Structure changes become cross-fades (blur + opacity). Morphs that explain structure still land, without overshoot.
- Banner: cross-fade + 0.2 s scale from 0.9 [D].
