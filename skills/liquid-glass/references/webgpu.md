# Liquid Glass on the Web (WebGPU, 2026)

Real Liquid Glass needs a GPU pass that reads the pixels *behind* the glass; CSS alone only
approximates it. Pick a path:

| Path | When | Status |
|---|---|---|
| **A. `liquid-rust-web`** | Any web app. Svelte / SolidJS: its kits (`bits-ui.md`); anything else: its framework-free API | Tested: liquid-rust 0.3.0, Chrome / Edge / WebView2 with WebGPU, CSS fallback elsewhere |
| **B. Raw WebGPU + `assets/*.wgsl`** | A JS / TS renderer of your own, no wasm | Same shaders; port the packing (§3) |
| **C. three.js WebGPURenderer / TSL** | Already in a three.js scene | Port `shader-math.md` to TSL nodes |
| **D. CSS only** | No WebGPU, or glass over live HTML | §5. `liquid-rust-web` does this for you as its CSS renderer |

WebGPU in 2026: Chrome / Edge 113+ (desktop, Android), Safari 26+ (macOS, iOS, iPadOS,
visionOS), Firefox 141+ on Windows, more later. Feature-detect `navigator.gpu` **and** a
device: headless browsers often have the API but no device.

## 1. The backdrop

Glass refracts only the picture you give it.

| Backdrop | How |
|---|---|
| Your own canvas / WebGPU scene / map | hand the canvas over; redraw the glass when it draws (`layer.wake()`) |
| Image | an address or an `<img>`; decoded once, blur built once |
| Video | the `<video>` element; copied per frame while it plays |
| Something you paint | a painter `(context, width, height) => void`, cached per size |
| Live DOM | **HTML-in-Canvas** (`<canvas content="drawable">` / `layoutsubtree`, then `drawElementImageToTexture`, older `copyElementImageToTexture`; Chrome origin trial, stable 157): feature-detect both names. Otherwise put the picture in the backdrop and the HTML on top |

Re-upload only when the backdrop really changes: still content is blurred once for the
whole screen and reused wherever glass moves.

## 2. Path A: `liquid-rust-web`

```bash
pnpm add github:al1h3n/liquid-rust#path:/js        # the npm package liquid-rust-web
```

```ts
import { createLayer, glass, SwitchDriver } from 'liquid-rust-web';
import 'liquid-rust-web/glass.css';

const { layer, top } = createLayer(document.querySelector('#stage')!, { backdrop: '/wallpaper.jpg', scheme: 'auto' });
const share = glass(layer, document.querySelector('#share')!, { kind: 'control', role: 'control' });
glass(layer, document.querySelector('#menu')!, { kind: 'panel', from: share });   // grows out of it
layer.configure({ reduceTransparency: 'off' });  // 'system' | 'on' | 'off'; same for reduceMotion, increaseContrast
layer.setView({ zoom: 1.1, blur: 8, dim: 0.2 }, 0.1);   // the backdrop steps back (an app opening)
```

What the core does: one WebGPU device and canvas per layer, every glass element synced to the
engine (resize, mutation, scroll, transitions, WAAPI, CSS transforms), pointers routed with
capture, labels coloured from the engine's ink, CSS renderer when the engine cannot run (the
layer says why: `layer.failure`), accessibility media queries with per-setting overrides,
tokens (`motions`, `glassKinds`) and drivers for switches, sliders, selections, sheets,
banners, swipe actions, a calendar, Genie and Dock magnification.

The engine alone, for a canvas app:

```ts
import { createEngine } from 'liquid-rust-web';
const result = await createEngine(canvas, devicePixelRatio, /* overlay */ false);
if (!result.ok) console.log(result.failure.reason);        // 'unsupported' | 'failed'
const engine = result.engine;
engine.setContent(image);                                 // image, canvas or video
const bar = engine.addContainer(16, 0);
const id = engine.add({ x: 24, y: 24, width: 44, height: 44, material: 'regular', role: 'control', container: bar });
// per frame: engine.update(dt) -> moving; engine.draw(); engine.readBackdrop()
```

Vite: `optimizeDeps: { exclude: ['liquid-rust-web'] }` (the wasm sits next to its module).
Size: about 290 KB of wasm, 120 KB gzipped; load it lazily when glass is optional.

Canvas colour: canvases are `bgra8unorm`; the binding renders through a `-srgb` view so the
shader's linear output is encoded. Overlay mode (`overlay: true`) draws only the glass, with
premultiplied alpha, on a transparent canvas above a page.

Windows note: with Transparency effects off, Chromium reports
`prefers-reduced-transparency: reduce` and the glass is drawn frosted. Offer an override.

## 3. Path B: raw WebGPU with the shared WGSL

Use `assets/glass.wgsl` and `assets/pyramid.wgsl` unchanged (synced with liquid-rust 0.3.0)
and port `liquid-rust/src/render/{pack,renderer,pyramid}.rs`.

**Pipelines** (`layout: "auto"`): `vs_glass` / `fs_glass` triangle strip, 4 vertices × N
instances, premultiplied blending (`one`, `one-minus-src-alpha`); `vs_full` / `fs_down_first`,
`fs_down` into `rgba16float`; `vs_full` / `fs_blit` into your target.

**Bindings** (group 0): 0 `Globals` uniform, 1 `shapes`, 2 `groups`, 3 sampler (linear,
clamp), 4 backdrop, 5 pyramid, 6 `materials` (storage, read-only), 7 `masks`
(`texture_2d_array<f32>`, `Rgba16Float`; one blank 1×1 layer when there are none, viewed as
an array even then).

```
Globals  (64 B):   viewport[w,h,1/w,1/h]  misc[pxPerPt, pyramidLevels, 0, 0]
                   light[sin a, -cos a, 0, 0]  touch[x px, y px, radius px, intensity]
Shape    (64 B):   center_half[cx,cy,hx,hy px]  params[corner extent px, shrink px, axisX, axisY]
                   stretch[along, 1/along, corner exponent, 0]
                   extra[bevel depth px, knob face, bevel profile n, mask layer + 1 (0 = box)]
Group    (48 B):   bounds[x0,y0,x1,y1 px]  info(u32)[firstShape, count, blend, 0]  geo[k px, presence, press, 0]
Material (144 B):  optics[refraction, face (0 dark..1 light), dispersion, frost σ px]
  one per shape    lighting[splay, specular, rim, rimShade]  edge[hairline, edgeBurn, contrast, dim]
                   shadow[offsetY px, σ px, opacity, body zoom]
                   light_a[lift, luminance, lumMix, lighten]  light_b[darkenTo, darken, 0, 0]
                   dark_a, dark_b (same)  tint[r, g, b (sRGB-encoded), strength]
```

`blend` = 1 when a group's members have different materials (mixed across the neck with the
smooth-min weights); 0 skips the loads. Effects are pre-multiplied by presence on the CPU.

**Per frame:** blit content → target; build the pyramid (scissored to the glass while the
content changes, once for the whole screen while it holds still); `draw(4, groupCount)` per
z-layer; glass on glass: `shader-math.md` §9.

**CPU side:** springs (`motion.md`), union-find clustering on the container spacing,
per-shape material resolve (size thickness, transparency slider, accessibility),
legibility (`liquid-rust/src/scene/legibility.rs`), no `writeBuffer` for unchanged bytes,
bind groups kept across frames.

## 4. Path C: three.js (WebGPURenderer)

Render the scene into a `RenderTarget`, build a pyramid (generated mips are box-filtered:
blockier frost unless you add the 13-tap pass), then an instanced quad whose `fragmentNode`
implements `shader-math.md` §1–8 in TSL. Keep glass in an orthographic overlay scene.

## 5. Path D: CSS

`liquid-rust-web`'s CSS renderer, so you only write this by hand outside it:

- **Frost and tone:** `backdrop-filter: blur(σ) saturate(s)` with the kind's σ from the tokens
  (`--lg-control-blur` 2.52 px, panel 6.72, clear 0.67, dock 0.42); a plain-fill face from
  `--lg-<kind>-face-light|dark`; `box-shadow` from `--lg-<kind>-shadow-*`; an inset top and
  bottom rim; a 0.5 px darkened outline (iOS 27 edge).
- **Lens (Chromium only):** `backdrop-filter: url(#lens) …` with an SVG `feImage` displacement
  map → `feGaussianBlur` → `feDisplacementMap` → `feColorMatrix saturate`, with
  `color-interpolation-filters="sRGB"` (or everything bends sideways). The map: per pixel the
  rounded-rect distance and normal, `t = clamp(−d / bevel, 0, 1)`, offset
  `refraction · bevel · tan(θ − asin(sin θ / 1.5))`, `θ = atan(slope(t, n))` with the same
  profile exponent as the shader (4 Regular, 2 Clear and Dock), sampled **inward**.
  `lens()` in `liquid-rust-web` builds it. Safari and Firefox ignore SVG backdrop filters:
  progressive enhancement only.
- **Squircle corners:** `corner-shape` (Chrome / Edge 139+): iOS's 60 % smoothing ≈
  `border-radius: calc(1.32 * R); corner-shape: superellipse(1.48)` inside
  `@supports (corner-shape: squircle)`; elsewhere `border-radius: R`.
- **Labels on glass:** `plus-darker` dark labels, `plus-lighter` light ones.

### Which path for which element

Glass over a picture (wallpaper, map, video) → the engine. Glass over HTML (a knob's lens
over its own track, a sheet over page text) → the engine still works if the HTML is drawn into
the backdrop; otherwise the CSS renderer. Glass that follows a finger every frame stays
synced: the core measures the element in the same frame.

### Headless kits

Bits UI and Kobalte expose state as attributes (`data-state`, `data-checked`,
`data-expanded`, `data-selected`, `data-highlighted`); `liquid-rust-web`'s CSS keys on all of
them, so both kits share one stylesheet. Keep content mounted through a closing morph
(`forceMount`) so the surface can fold back into its control.

## 6. Web rules

- Canvas in device pixels; scene units are CSS pixels; pass `devicePixelRatio` (capped, 2 is
  plenty) as the scale.
- Render on demand: the loop sleeps when nothing moves, unless the backdrop is a playing
  video or a scrolling canvas.
- Pointers through `setPointerCapture`; `touch-action: none` on draggable glass.
- Text stays in the DOM above the canvas, for accessibility and selection.
- One device per page where you can; never a context per glass element.
- **Glass over DOM content is CSS.** The engine's canvas is under the page: an opaque element
  (a window, a card) hides any engine glass under it, and engine glass cannot cover it. Glass
  inside an opaque surface, and glass that floats over one (Spotlight, Control Center, menus
  over a window), uses the CSS renderer (no layer in its context); it blurs what is really
  under it.
- **Portals wait for the layer.** A surface created before its layer has mounted must not
  resolve its portal yet, or it lands in `<body>` and is placed against the browser window,
  outside the layer (an alert pinned to the page's corner). Render the portal once
  `layer.top` exists.
- **Continuous corners in CSS.** `corner-shape: superellipse(1.6)` (Chrome 139+, inside
  `@supports`) on every rounded rectangle of the page (windows, sheets, cards, rows, icon
  tiles) so DOM outlines match the engine's squircle glass; never on capsules or circles.
- **Scroll bars** as Apple draws them: a thin rounded thumb (11 px box, 3 px inset), no track,
  no arrows (`::-webkit-scrollbar*`).
- **Glass over page content** (a menu over a photo, an overlay over a window) is CSS glass: give
  it no layer in its context. Engine glass refracts only the layer's backdrop.
- **Clip the stage.** DOM content that leaves a stage keeps drawing; the canvas glass does not.
  `overflow: clip` on the stage keeps both together; keep draggable demos inside it.
