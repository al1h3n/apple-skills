# Liquid Rust: the engine

Crate [`liquid-rust`](https://crates.io/crates/liquid-rust) **0.2.5** (source:
`github.com/al1h3n/liquid-rust`, docs: docs.rs/liquid-rust), on wgpu 30, edition 2024,
Rust 1.99+. It is a library that works in any project: native (Vulkan, Metal, DX12, GL) and
the web (WebGPU via wasm, `web/`). Its shaders are the ones in this skill's `assets/`.

```toml
[dependencies]
liquid-rust = "0.2.5"
```

## When to use it

- Rust apps on any GPU UI stack (winit, egui/iced custom painters, Bevy UI overlays, Tauri
  canvases) that need real Liquid Glass.
- Web pages through `wasm-pack`, see `webgpu.md` §2.
- Not inside native Apple apps: use `glassEffect` there (`native-apple.md`).

## API in one screen

```rust
use liquid_rust::*;   // Scene, Glass, Rect, Shape, Material, Color, Tone, Shadow,
                      // Appearance, Spring, Animated, Container(Id), GlassId, Renderer, Frame, Vec2

let mut scene = Scene::new();
scene.appearance = Appearance { dark: false, transparency: 0.5, ..Default::default() };

let bar = scene.add_container(14.0, 0);                  // spacing pt (merge distance), z
let a = scene.add(Glass::new(Rect::new(24.0, 24.0, 52.0, 52.0)).interactive().container(bar));
let lens = scene.add(Glass::new(Rect::new(40.0, 300.0, 90.0, 90.0)).material(Material::clear()).lens());
let ok = scene.add(Glass::new(Rect::new(300.0, 24.0, 110.0, 52.0)).interactive());
scene.add(Glass::new(Rect::new(20.0, 600.0, 360.0, 300.0)).shape(Shape::Rounded(34.0)).z(1)); // glass over glass

// Layout
scene.set_frame(a, Rect::new(24.0, 90.0, 250.0, 300.0));  // morph (snappy spring)
scene.set_frame_with(a, r, Spring::new(0.45, 0.25));       // custom spring
scene.place(a, r);                                         // jump, no spring (window resize, scroll)
scene.set_shape(a, Shape::Rounded(28.0));                  // instant outline change
scene.set_container_spacing(bar, 20.0);                    // 0 = no merging
scene.remove(a);                                           // dematerialize, then dropped

// Material
scene.set_material(ok, Material::regular());               // instant (cancels a running fade)
scene.set_material_with(ok, Material::tinted(Color::BLUE), Spring::new(0.3, 0.0)); // fade

// Grow one glass out of another, and back
let results = scene.expand_from(a, Glass::new(r).shape(Shape::Rounded(28.0)), Spring::new(0.5, 0.2));
scene.collapse_into(results, a, Spring::new(0.35, 0.0));

// Input and light
scene.set_light_angle(45.0);                               // device tilt / focus sweep
scene.pointer_down(p); scene.pointer_move(p); scene.pointer_up();   // points; down returns Option<GlassId>
scene.hit_test(p);                                         // topmost interactive glass or lens
scene.current_frame(a); scene.current_corner(a);           // on screen now, mid-animation

// Switches
scene.physics = false;                                     // every change lands on the next update
scene.glass_on_glass = false;                              // Apple's "glass can't sample glass"
scene.appearance.adapt_to_content = false;                 // small glass follows the scheme

// Frame
let mut renderer = Renderer::new(&device, surface_format); // once per target format
let animating = scene.update(dt);                          // false → stop requesting frames
renderer.render(&device, &queue, &mut encoder,
    &Frame { content, target, size, scale, content_changed }, &scene);
```

Shapes: `Shape::Capsule`, `Shape::Rounded(r)` (Apple continuous squircle, smoothing
`IOS_SMOOTHING` = 0.6, the default for panels), `Shape::Circular(r)` (quarter circles), and
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
| `Appearance` | `dark`, `transparency` (iOS 27 slider), `reduce_transparency`, `increase_contrast`, `reduce_motion`, `adapt_to_content` |
| `Spring` / `Animated` | Apple's duration/bounce springs (`SMOOTH`, `SNAPPY`, `BOUNCY`, `INTERACTIVE`), solved analytically; bounce is clamped to ±0.95 so everything comes to rest |
| `Container` | `GlassEffectContainer`: spacing = merge distance, z = layer |
| `Renderer` / `Frame` | wgpu passes; one renderer per target format (and per pass with `render_layers`) |

## Behaviour to know (0.2.x)

- **Fused glass keeps its own material (0.2.5).** Each shape in a container carries its own
  material, resolved for its own size; across the liquid neck the materials blend with the
  same smooth-min weights as the outline. A tinted button pressed against a plain one stays
  tinted; a thin bar fused to a big panel stays thin. Before 0.2.5 the whole group took the
  material of the glass added first, so don't work around that any more.
- **Material fades (0.2.5).** `set_material_with(id, m, spring)` fades every knob, and a tint
  fades in by its strength (never through black). Interrupting carries on from what is on
  screen. Use it for a primary button's tint appearing once the action becomes available.
- **Transparency slider (0.2.4).** Toward "fully tinted", the dark face goes to near-black
  `#1C1C1E` at about 70–85 % and the light face to white at about 70–88 %; nothing is pulled
  toward mid grey any more. Apps that capped `transparency` at 0.5, or darkened/whitened their
  own panes to hide the grey, can drop that workaround. Reduce Transparency uses the same
  tinted faces plus heavy frost.
- **Adaptivity.** Regular glass always takes the light *or* the dark face, never a 50/50 grey
  blend. Small glass (buttons, tab bars) flips with the content; from about 180 pt (menus,
  panels) it follows the colour scheme. `appearance.adapt_to_content = false` keeps small
  glass on the scheme too (labels that follow the scheme).
- **Glass on glass.** By default a higher z-layer refracts the glass that is really under it,
  rebuilt only where layers overlap. `scene.glass_on_glass = false` follows Apple's rule:
  every layer refracts the content alone and covers the glass beneath it (menus over
  toolbars). Presses always go to the topmost glass.
- **Labels under upper glass.** Labels drawn after `render` are on top of everything. When an
  upper layer (a lens over a menu) should bend the labels of lower layers, split the frame:

  ```rust
  lower.render_layers(&device, &queue, &mut encoder, &Frame { content: &content_view, target: &middle_view, size, scale, content_changed }, &scene, ..2);
  // draw the labels of layers 0 and 1 into `middle`
  upper.render_layers(&device, &queue, &mut encoder, &Frame { content: &middle_view, target: &target_view, size, scale, content_changed: true }, &scene, 2..);
  ```

  Use one `Renderer` per pass; each keeps the blur of its own content.
- **Merge distance.** Below half the container spacing shapes join with a liquid neck; at the
  full spacing they are clean. Rest shapes either under half (joined) or at least the full
  spacing apart: a neck that breaks just as motion stops reads as a glitch.
- **Physics off vs Reduce Motion.** `scene.physics = false` makes every change land on the next
  `update` (no springs, press growth, lens lift or stretch): static UI, tests, layout work.
  `appearance.reduce_motion` is gentler: it keeps the motion and drops the bounce.
- **Bad input is absorbed.** A `0 × 0` render size is skipped; NaN `dt` counts as zero; a NaN
  or infinite coordinate leaves that value where it was; springs at large values (thousands of
  points) still come to rest, so `scene.update` always ends up returning `false`.

## Frame contract

- `content` and `target` are different textures of the same size. `content` must be readable
  as linear colour: an sRGB view (`Rgba8UnormSrgb` / `Bgra8UnormSrgb`) or a float format.
  On the web, configure the canvas with `viewFormats: ["bgra8unorm-srgb"]` and render through
  that view (the wasm wrapper does this).
- Set `content_changed: false` while the content holds still: its blur pyramid is built once
  for the whole screen and reused wherever the glass moves, including under glass on glass.
- The scene is in points, top-left origin. `Frame::scale` converts points to pixels.
- Draw glyphs and labels after `render`, with vibrancy (additive / plus-lighter for light text,
  multiply / plus-darker for dark text). Don't put glass on glass for that.

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
| One material per blob | Per-shape materials blended across the liquid neck |
| Alpha-ish fades | Materialize by ramping lensing; neighbours grow out of each other |
| Glass on glass not handled | Correct layered backdrops, rebuilt only where layers overlap, or Apple's no-sampling mode |

## Tuning recipes

- **Apple Control Center tile:** `Material::clear()` at 70×70 or 155×155 with `Rounded(30)`,
  over a dimmed background (35 % black plus blur under the whole panel).
- **Dock / tab bar over a wallpaper:** `Material::clear_bar()` with `Rounded(38)` at 103 pt height.
- **Toolbar cluster:** `Material::regular()` capsules 44–52 pt in a container whose spacing is
  larger than the gap between them, so they share one liquid body. A tinted primary button can
  sit in the same container: it keeps its tint while fused.
- **Primary action becoming available:** add it as `regular()`, then
  `set_material_with(id, Material::tinted(Color::BLUE), Spring::new(0.3, 0.0))`.
- **Menu from a button:** `expand_from(button, menu_glass, Spring::new(0.5, 0.2))` (same
  container, starts fused and with the button's outline), close with
  `collapse_into(menu, button, Spring::new(0.35, 0.0))`. End the gap at least the full spacing
  away for a clean split. Set `glass_on_glass = false` if the menu overlaps other glass.
- **Search bar opening into results:** same as the menu, with a `Rounded(28)` results panel.
  Clip the results list to `current_frame` / `current_corner` so content ends exactly where
  the glass does mid-morph.
- **Slider knob / tab selection lens:** `.lens()` glass at a higher z than the track. It lifts
  ×1.3 and stretches with velocity while dragged. If you want Apple's resting state (a plain
  fill), drop it back to a fill when released.
- **Matching a Figma design:** copy Refraction/100, Depth, Dispersion/100, Frost, Splay/100
  and Light% into `Material`. Copy fills into `Tone`, and shadows into `Shadow` / `edge` / `rim`.

## Numbers

- 0.7 ms per frame over static content, 0.8 ms over content that changes every frame, at
  1206×2622 for 8 groups with merging and a glass-on-glass layer (RTX 3060 Ti). Recording a
  frame costs about 64 µs of CPU: buffers are written only when they change and bind groups
  are kept across frames.
- Tests: springs (frame-rate independence, overshoot, runaway bounce, large values coming to
  rest), materials (slider faces, fades), scene (hit-test, drag, stretch, reduce motion,
  physics off, expand/collapse), clustering, GPU render tests (compositing contract, stacked
  layers, backdrop cache, fused materials; skipped without an adapter) and naga validation of
  both shaders. Clippy is clean on native and wasm.
- `snapshot` example modes: `lock`, `home` (Apple kit positions for A/B comparison),
  `corners`, `showcase`, `press`, `search`, `bench` (`LIQUID_BENCH_FRAMES` lowers the frame
  count on software renderers).
