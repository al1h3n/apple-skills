# Liquid Rust: the engine

Crate [`liquid-rust`](https://crates.io/crates/liquid-rust) **0.3.0** (source
`github.com/al1h3n/liquid-rust`, docs docs.rs/liquid-rust). wgpu 30, edition 2024, Rust 1.99+.
Native (Vulkan, Metal, DX12, GL) and the web (WebGPU through wasm). Its shaders are this
skill's `assets/`. Web packages on top: `bits-ui.md`.

```toml
liquid-rust = "0.3"                                          # scene + renderer + kit
liquid-rust = { version = "0.3", default-features = false }  # scene only: no wgpu, testable anywhere
```

Features: `render` (the wgpu `Renderer`), `kit` (headless components). Both on by default.

## When

- Rust apps on any GPU stack (winit, egui / iced painters, Bevy overlays, Tauri canvases).
- Web pages: through `liquid-rust-web` (`bits-ui.md`), not by hand.
- Not native Apple apps: `glassEffect` there (`native-apple.md`).

## Two halves

| | Needs | Holds |
|---|---|---|
| `Scene` | `glam`, `slotmap` | glass, containers, pointers, springs, morphs, legibility, display list |
| `Renderer` | wgpu 30 | blur pyramid, glass shader, glass-on-glass layers, backdrop read-back |

A renderer of your own can draw `scene.drawables(.., &mut list)` (`Drawables`: shapes,
groups, light, glow, masks).

## API in one screen

```rust
use liquid_rust::*;

let mut scene = Scene::new();
scene.appearance = Appearance { dark: false, transparency: 0.5, ..Default::default() };
scene.feel = Feel::V27;                                   // V26: the louder release, for A/B

let bar = scene.add_container(16.0, 0);                   // spacing = merge distance, z
let b = scene.add(Glass::new(Rect::new(24.0, 24.0, 44.0, 44.0)).interactive().container(bar));
let cta = scene.add(Glass::new(r).material(Material::tinted(Color::BLUE)).interactive());
let panel = scene.add(Glass::new(r).shape(Shape::Rounded(34.0)).material(Material::panel()).z(1));

// Motion
scene.place(b, r);                                        // jump: layout, scroll, resize
scene.set_frame_launched(b, r, Spring::EMERGE);           // a tap sent it: full speed, exponential landing
scene.set_frame_with(b, r, Spring::SMOOTH);               // a change nobody sent
scene.fling(sheet, r, velocity, Spring::FLICK);           // let go mid-flick
let menu = scene.morph_from(b, Glass::new(r).shape(Shape::Rounded(28.0)).z(1), Spring::EMERGE); // one shape
scene.shrink_into(menu, b, Spring::RECEDE);               // back into its button, dropped on landing
scene.expand_from(field, results, Spring::new(0.5, 0.2)); // two shapes joined by a liquid neck
scene.collapse_into(results, field, Spring::new(0.35, 0.0));
scene.set_present(id, false);                             // materialize / dematerialize by lensing
scene.remove(id);

// Material, shape, light
scene.set_material_with(cta, Material::tinted(Color::BLUE), Spring::new(0.3, 0.0)); // fade (never through black)
scene.set_shape(b, Shape::Rounded(28.0));
scene.set_light_angle(45.0);

// Input: several pointers at once
scene.pointer_down(PointerId::MOUSE, p);                  // -> Option<GlassId>
scene.pointer_move(PointerId::MOUSE, p);                  // a held control leans after the finger
scene.pointer_up(PointerId::MOUSE);
scene.hit_test(p);

// Legibility: the host asks, never guesses
scene.set_backdrop_rgba8(area, w, h, &pixels);            // or renderer.read_backdrop(&device, &mut scene)
scene.ink_of(id);                                         // Ink::Dark | Ink::Light for labels
scene.content_of(id);                                     // frame, corner, ink, opacity, softness for content
scene.luma_under(id);                                     // dim bright media under Clear

// Frame
let moving = scene.update(dt);                            // false: stop asking for frames
renderer.render(&device, &queue, &mut encoder,
    &Frame { content, target, size, scale, content_changed, overlay: false }, &scene);
let reading = renderer.read_backdrop(&device, &mut scene); // true while a read-back is in flight
```

## Types

| Type | Role |
|---|---|
| `Glass` | builder: `.shape()`, `.material()`, `.interactive()`, `.role(Role)`, `.legibility(Legibility)`, `.container(id)`, `.z(n)` |
| `Shape` | `Capsule`, `Rounded(r)` (continuous, 60 % smoothing), `Circular(r)`, `Smooth { radius, smoothing }`, `Mask(MaskId)` |
| `Material` | Figma GLASS knobs + `profile` (bevel exponent: 4 squircle Regular, 2 quarter circle Clear / Dock) + `dim`, `magnify`, `tint`. Presets `regular()` (thickens into the panel by size), `panel()`, `clear()`, `dock()`, `lens()`, `tinted(c)`, `identity()` |
| `Role` | `Surface` (no input), `Control`, `Lens { press, slide }`, `Knob { slide }` |
| `Legibility` | `Auto`, `Backdrop`, `Scheme`, `Light`, `Dark` |
| `Appearance` | `dark`, `transparency`, `reduce_transparency`, `increase_contrast`, `reduce_motion`, `adapt_to_content` |
| `Spring` | closed form; `SMOOTH`, `SNAPPY`, `BOUNCY`, `INTERACTIVE`, `EMERGE` (0.45 s), `RECEDE` (0.3 s), `FLICK`, `ARRIVE`, `KNOB_SETTLE`, `KNOB_TRAVEL` |
| `Motion::ALL` | 20 named motions with their spring and whether launched |
| `Feel` | what a finger does: press growth (+5 pt, cap 15 %), launch times, glow out, lens lift, knob lift, `press_pull` lean, `pull_release`. `V27`, `V26` |
| `MaskField` / `MaskId` | a baked silhouette (512-texel signed distance + normal) |

## Mask glass

```rust
let digits = scene.add_mask(&alpha, width, height);        // 128+ is inside; bake once
scene.add(Glass::new(frame).shape(Shape::Mask(digits)).material(Material::clear()));
```

Bends, lights, shadows and fuses like any glass. The frame should have the mask's aspect;
presses hit the frame. `MaskField::from_alpha` bakes off-thread; pass an `Arc<MaskField>` to
`add_mask_field` to share a bake between scenes (the renderer then skips the upload). Native
only so far (no web call). Uses: Lock Screen digits, glyph glass, app icons (LIGMA).

## Kit (feature `kit`)

Headless: the engine owns glass and choreography, the host draws the rest where `slots()` /
`content()` say. Sizes in `kit::metrics::{ios, macos}`.

| Component | Engine | Host draws |
|---|---|---|
| `Switch`, `Slider` | knob, lens while held, taps, drags, steps | track and fill |
| `Selection` | segmented / tab selection: slides, lifts into a lens | items; the fill at rest |
| `Menu` | surface out of its control and back (`morph_from` / `shrink_into`) | rows |
| `Sheet` | detents, follows the finger, carries its speed, thrown away | content |
| `Banner` | arrives from where notifications live, leaves there | content |

```rust
let mut switch = kit::Switch::add(&mut scene, origin, metrics::ios::SWITCH_TRACK, metrics::ios::KNOB, false, 0);
if switch.pointer_down(&mut scene, pointer, p) { /* took it */ }
if let Some(on) = switch.pointer_up(&mut scene, pointer) { /* changed */ }
```

A button is glass with `.interactive()`; alerts and popovers are `panel()`. `Material::dome()`
is a held point as a magnifying dome (no flat top, minified middle, mirrored rim).

Web core extras: `magnifyDock(items, x, most = 2.4, reach = 3)` (layout magnification with a
fast spring; size items with `--lg-magnify`), `genie` (strips mapped onto exact quads,
`stripMatrix`), `dragWindow(event, offset, move)`, `stackBanner(element)` (banners open
together pile up), `fillTint(rgb, dark)` (a flat fill drawn as tinted glass at its exact
colour). Kit `Window`: `sidebar` (flush with the window, macOS 27; `inset` floats it, macOS 26),
`movable` (drag the bar, double-click zooms), `offset`; glass inside it is CSS; the scroll
edge band shows only while content is under the toolbar. Kit `Banner`: banners open together
stack (newest in front, older ones smaller behind it), swipe up or × closes, `onactivate`
opens, `actions` puts buttons along the bottom (Mac). The layer marks
`data-lg-transparency="clear" | "tinted"` so the CSS renderer follows the slider's ends.

## Behaviour to know

- **Launched motion.** Finger-sent things leave at full speed and only slow (`launch_to`);
  the first frame covers a fifth or more of the way. No ease-in.
- **One shape.** While a `morph_from` surface is out, its source is not drawn; it lands back
  without a pixel changing. A source fused to neighbours leaves their union on frame 1: rest
  such buttons a full spacing apart.
- **Lean.** A held control follows a sliding finger with a rubber band (shift 0.6 of the band,
  a little stretch) and springs home on release (`Feel::press_pull`, `pull_release`).
- **Interrupted fades** keep both ends (`FadeEnd`): reversing mid-fade carries on from the screen.
- **Fused glass keeps its own material** per shape, blended across the neck.
- **Adaptivity.** Regular takes the light or the dark face, never grey. Small glass flips with
  content (to light instantly, to dark after 0.1 s), from about 180 pt it follows the scheme;
  a container decides together.
- **Glass on glass.** Default: upper layers refract the glass under them, rebuilt only where
  layers overlap. `scene.glass_on_glass = false`: Apple's "glass can't sample glass".
- **Labels under upper glass:** split the frame with `render_layers(.., ..2)` → labels →
  `render_layers(.., 2..)`, one `Renderer` per pass.
- **Physics off** (`scene.physics = false`): everything lands on the next update (tests,
  static UI). **Reduce Motion**: keeps motion, drops bounce, press growth and lean.
- **Overdrag and motion blur.** `Slide::pull(share)`: how far a knob follows past an end, as
  a share of its width (0.2 default; about 1 follows the pointer off the track, macOS 27).
  `Glass::motion_blur(seconds)` / `scene.set_motion_blur`: a moving glass is drawn with soft
  edges along its path (1/60 s for a flung knob); none when still or with Reduce Motion.
  `kit::Slider::set_style(scene, pull, motion_blur)`; web `partStyle`; kit `Slider pull
  motionBlur` props.
- **The person's own amounts.** `Appearance::bend` and `::blur` scale every material's
  refraction and frost (1 as designed, 0 to 3); `Appearance::tint` colours every glass that
  has no tint of its own (about 0.25 at the default transparency, 0.75 at Tinted, never a
  lens). Web: `setAppearance({ bend, blur, tint })`; layer settings `bend`, `blur`, `tint`;
  kit `GlassLayer bend glassBlur tint`. Labels on a strongly tinted face take the ink of the
  tint (`inkOn`), so text stays readable; CSS glass that is frosted or fully tinted follows
  the scheme.
- **Lenses stay clear** under Reduce Transparency (`Material::lens`, `Material::dome`): it
  frosts and fills every other glass, never a held knob over its own track.
- **Bad input absorbed:** 0 × 0 frames skipped, NaN dt = 0, springs at huge values still rest.

## Frame contract

- `content` and `target`: different textures, same size. `content` readable as linear colour
  (an `*Srgb` view or float). On the web the wasm binding configures this.
- `content_changed: false` while content holds still: its blur is built once for the whole
  screen and reused wherever glass moves.
- `overlay: true`: only the glass, on a transparent target (a canvas over a page).
- Points, top-left origin; `Frame::scale` = pixels per point.
- Draw labels after `render`, with vibrancy, coloured from `ink_of`.

## Tokens

`cargo run --example export_tokens --no-default-features -- out/` writes `tokens.css` /
`tokens.ts`: every motion as a CSS `linear()` easing with its settle time, the feel numbers,
each glass kind as blur, plain-fill face and shadow. CSS that animates beside the engine
stays in step.

## Recipes

- **Control Center tile:** `clear()`, 70×70 or 155×155, `Rounded(30)`, over 35 % dim + blur.
- **Dock / tab bar over a wallpaper:** `dock()`, `Rounded(38)`, 103 pt.
- **Toolbar cluster:** `regular()` capsules in one container, spacing larger than their gaps.
- **Primary action becoming available:** add `regular()`, fade to `tinted` with `set_material_with`.
- **Menu from a button:** `kit::Menu::open` (or `morph_from` + `shrink_into`).
- **Search bar into results:** `expand_from` / `collapse_into`; clip results to `content_of`.
- **Figma design:** Refraction/100, Depth, Dispersion/100, Frost, Splay/100, Light % →
  `Material`; fills → `Tone`; shadows → `Shadow`, `edge`, `rim`.

## Numbers

- 0.7–0.8 ms a frame at 1206×2622, 8 groups, merging and a glass-on-glass layer (RTX 3060 Ti);
  idle = 0 frames.
- Tests: 155 unit (springs, materials, scene, input and lean, legibility, kit, masks),
  22 GPU (compositing contract, layers, caches, fused materials, mask glass; skipped without
  an adapter), naga on both shaders, 28 doc tests. Clippy clean native and wasm.
- `snapshot` example modes: `lock`, `home`, `corners`, `showcase`, `press`, `search`, `bench`,
  `mask`. `swatch`: one material over an image.
