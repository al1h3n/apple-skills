# Liquid Rust: the engine

`C:\Users\al1h3n\Downloads\LiquidRust`, crate `liquid-rust`, on wgpu 30 and edition 2024.
It is a library that works in any project: native (Vulkan, Metal, DX12, GL) and the web (WebGPU via wasm, `web/`).
Its shaders are the ones in this skill's `assets/`.

## When to use it

- Rust apps on any GPU UI stack (winit, egui/iced custom painters, Bevy UI overlays, Tauri
  canvases) that need real Liquid Glass.
- Web pages through `wasm-pack`, see `webgpu.md` §2.
- Not inside native Apple apps: use `glassEffect` there (`native-apple.md`).

## API in one screen

```rust
use liquid_rust::*;

let mut scene = Scene::new();
scene.appearance = Appearance { dark: false, transparency: 0.5, ..Default::default() };

let bar = scene.add_container(14.0, 0);                  // spacing pt, z
let a = scene.add(Glass::new(Rect::new(24.0, 24.0, 52.0, 52.0)).interactive().container(bar));
let lens = scene.add(Glass::new(Rect::new(40.0, 300.0, 90.0, 90.0)).material(Material::clear()).lens());
scene.add(Glass::new(Rect::new(300.0, 24.0, 110.0, 52.0)).material(Material::tinted(Color::BLUE)).interactive());
scene.add(Glass::new(Rect::new(20.0, 600.0, 360.0, 300.0)).shape(Shape::Rounded(34.0)).z(1)); // glass over glass

scene.set_frame(a, Rect::new(24.0, 90.0, 250.0, 300.0));  // morph (snappy spring)
scene.set_frame_with(a, r, Spring::new(0.45, 0.25));       // custom spring
scene.remove(a);                                           // dematerialize, then dropped
scene.set_light_angle(45.0);                               // device tilt / focus sweep
scene.pointer_down(p); scene.pointer_move(p); scene.pointer_up();   // points

let animating = scene.update(dt);                          // false → stop requesting frames
renderer.render(&device, &queue, &mut encoder,
    &Frame { content, target, size, scale, content_changed }, &scene);
```

Shapes: `Shape::Capsule`, `Shape::Rounded(r)` (Apple continuous squircle, smoothing 0.6, the
default for panels), `Shape::Circular(r)` (quarter circles), and
`Shape::Smooth { radius, smoothing }` (any Figma smoothing 0–1). `smooth_corner(r, s, budget)`
exposes the Figma-to-superellipse mapping for your own drawing or hit-testing.

Drawing plain (non-glass) squircles is a different job: vector paths for SVG, icons or a 2D
renderer. Don't add it to Liquid Rust, and don't start a new crate for it either:
`figma_squircle` (Figma-exact SVG paths) and `smooth-frame` (Sketch-style Bézier paths)
already exist on crates.io. Liquid Rust only needs the distance field, and that lives in
`glass.wgsl` / `scene.rs`.

| Type | Role |
|---|---|
| `Material` | All knobs, named after Figma's GLASS panel. Presets: `regular()`, `clear()`, `clear_bar()`, `tinted(c)`, `identity()`, and `.tint(c)` |
| `Tone` | the kit's "Fill + Shadow" fills (lift / luminosity / lighten / darken) |
| `Appearance` | dark scheme, the iOS 27 transparency slider, Reduce Transparency / Increase Contrast / Reduce Motion |
| `Spring` / `Animated` | Apple's duration/bounce springs (`SMOOTH`, `SNAPPY`, `BOUNCY`, `INTERACTIVE`), solved analytically |
| `Container` | `GlassEffectContainer`: spacing = merge distance, z = layer |
| `Renderer` / `Frame` | wgpu passes; one renderer per target format |

## What it does that the old `liquid-claude` didn't

| liquid-claude | Liquid Rust |
|---|---|
| Explicit Euler springs plus constant friction, so results depend on frame rate and jitter | Closed-form damped springs; same curve at 60/120/165 Hz; interruptible |
| Circle-radius collision pushes between pills ("buggy physics") | No collisions. Merge and split come from the SDF smooth union, as Apple does it |
| Full-screen pass looping 24 pills per pixel | One instanced quad per merged group; loops only that group's shapes |
| `dFdx` normals on the smooth union (2×2-quad noise) | Analytic gradient through the smooth union, diagonal-softened |
| 5-tap blur re-sampling a procedural background | Shared blur pyramid, B-spline continuous radius, region-limited, cached |
| Uniform white rim + sine shimmer | Kit-calibrated lit line (light angle, splay), inner rims, iOS 27 burn edge |
| Tint = `glass × tint` | OKLab stained glass that follows backdrop lightness |
| Alpha-ish fades | Materialize by ramping lensing; neighbours grow out of each other |
| Glass on glass not handled | Correct layered backdrops, rebuilt only where layers overlap |

## Tuning recipes

- **Apple Control Center tile:** `Material::clear()` at 70×70 or 155×155 with `Rounded(30)`,
  over a dimmed background (35 % black plus blur under the whole panel).
- **Dock / tab bar over a wallpaper:** `Material::clear_bar()` with `Rounded(38)` at 103 pt height.
- **Toolbar cluster:** `Material::regular()` capsules 44–52 pt in a container whose spacing is
  larger than the gap between them, so they share one liquid body.
- **Menu from a button:** add glass at the button frame in the same container, then
  `set_frame_with(menu, menu_rect, Spring::new(0.45, 0.25))`. To close, `set_frame` it back and `remove`.
- **Slider knob / tab selection lens:** `.lens()` glass at a higher z than the track. It lifts
  ×1.3 and stretches with velocity while dragged. If you want Apple's resting state (a plain
  fill), drop it back to a fill when released.
- **Matching a Figma design:** copy Refraction/100, Depth, Dispersion/100, Frost, Splay/100
  and Light% into `Material`. Copy fills into `Tone`, and shadows into `Shadow` / `edge` / `rim`.

## Numbers

- 0.6–0.7 ms per frame at 1206×2622 for 8 groups with merging and a glass-on-glass layer (RTX 30-series).
- 27 tests: springs (frame-rate independence, overshoot, analytic velocity), materials, scene
  (hit-test, drag, stretch, reduce motion), clustering, and 2 GPU render tests. Clippy is clean
  on native and wasm.
- `snapshot` example modes `lock` and `home` re-render Apple's kit screens for A/B comparison.
