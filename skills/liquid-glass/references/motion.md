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
  frames.

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

## 2. Interaction choreography

Values marked [Apple] are described in WWDC25; numbers are [Derived] from frame-stepping
iOS 26/27 recordings and tuned in Liquid Rust — tune by eye, keep the *shape*.

| Moment | What moves | Spring |
|---|---|---|
| **Press down** (`.interactive`) | scale 1 → 1 + min(0.15, 10 pt / size); glow intensity 0 → 1 starting at the touch point; glow radius 0 → 1.2× element size; light "energizes" (specular ×1.6) [Apple: flexes and energizes with light] | scale `0.25 s, 0`; glow `0.2 s, 0`; radius `0.45 s, 0` |
| Glow spill | the glow is a scene light, so nearby glass in the same region lights up too [Apple] | — |
| **Release** | scale → 1, glow → 0 | scale `0.5 s, 0.35` (one visible overshoot); glow `0.6 s, 0` |
| **Pointer hover** (macOS/iPad) | scale 1 → 1.03, specular ×1.2 | `0.3 s, 0` |
| **Drag a lens / knob** | position follows finger; **stretch along velocity** (area-preserving), lens lifted: scale 1 → ~1.3, refraction stronger (magnifier over the value) [Apple: "elements can lift up into Liquid Glass temporarily"] | follow `0.15 s, 0`; stretch `0.35 s, 0.4` (jelly settle when the finger stops); lift `0.3 s, 0.15` |
| **Morph / layout** (glassEffectID) | frame x, y, w, h and corner radius as independent springs; neighbours fuse/split through the SDF union automatically | `0.5 s, 0.15` |
| **Menu / popover from button** | the button frame springs into the menu frame; thickness (frost, shadow, face opacity) ramps up with size [Apple: "bubble simply pops open"] | frame `0.45 s, 0.25`; content fade-in delayed ≈0.08 s |
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

## 3. Frame pacing

- Drive from the display: `requestAnimationFrame` (web), `CADisplayLink` (iOS),
  `winit` `RedrawRequested` + `request_redraw()` (native Rust). Use the frame timestamp
  delta, clamp to ≤ 1/30 s to avoid explosions after a stall (analytic springs are stable
  anyway, the clamp is for visual continuity).
- **Render on demand**: when every spring is at rest and the backdrop hasn't changed, stop
  requesting frames. Liquid Rust's `Scene::update(dt) -> bool` returns whether anything is
  still animating.
- Present with `PresentMode::Fifo` (vsync) or `Mailbox`; on ProMotion/120 Hz screens the
  analytic springs are automatically smooth.
- Keep CPU work per frame O(shapes) and allocation-free; upload one small storage buffer.

## 4. Reduce Motion

- Every spring: bounce → 0 (critically damped), duration ×0.8.
- No stretch, no press scale, glow intensity ×0.5, light sweep disabled.
- Morphs still happen (they convey structure) but without overshoot.
