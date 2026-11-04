# Liquid Glass on the Web (WebGPU, 2026)

You can't get real Liquid Glass from CSS alone. It needs a GPU pass that reads the pixels
*behind* the glass. Pick a path:

| Path | When | Status |
|---|---|---|
| **A. Liquid Rust → wasm** | You can ship a ~1 MB wasm, or the page is a canvas app | Tested: `liquid-rust/web` 0.2.5 (wasm-pack, WebGPU) |
| **B. Raw WebGPU + `assets/*.wgsl`** | Pure JS / TS stack, no wasm | Same shaders; port `renderer.rs` packing (§3) |
| **C. three.js WebGPURenderer / TSL** | Already in a three.js scene | Port the maths in `shader-math.md` to TSL nodes |
| **D. CSS fallback** | WebGPU missing, or for DOM chrome where you can't capture the backdrop | §5. An approximation only |

WebGPU availability in 2026: Chrome/Edge 113+ (desktop and Android), Safari 26+ (macOS, iOS,
iPadOS, visionOS), Firefox 141+ on Windows and later on more platforms. Always feature-detect
`navigator.gpu` and fall back to D.

## 1. Getting the backdrop into a texture

The glass can only refract what you give it.

| Backdrop | API |
|---|---|
| Your own canvas/WebGPU scene | render it into a texture with `TEXTURE_BINDING` (best) |
| Image | `createImageBitmap(blob, {resizeWidth, resizeHeight})` → `queue.copyExternalImageToTexture` |
| Video | `device.importExternalTexture({source: video})` per frame (zero-copy), or copy per frame |
| Live DOM | **HTML-in-Canvas** (Chrome origin trial through 160, planned default ~157). Put `<canvas content="drawable">` (earlier builds: `layoutsubtree`) around the DOM, then call `queue.drawElementImageToTexture({source: el, …}, {texture, size})` in the canvas `paint` event (older builds: `copyElementImageToTexture`). The API is still moving, so feature-detect both names |
| Arbitrary DOM, no HTML-in-Canvas | snapshot with html2canvas (static) or keep the glass over a canvas-rendered region only |

Set `content_changed` / re-upload only when the backdrop really changes. With a static
wallpaper the blur pyramid is built once.

## 2. Path A: Liquid Rust in the browser (recommended)

```bash
git clone https://github.com/al1h3n/liquid-rust && cd liquid-rust/web
wasm-pack build --target web --release
python -m http.server
```

Then open `/index.html?bg=img.png`. The JS API (see `web/index.html`):

```js
import init, { LiquidGlass } from "./pkg/liquid_rust_web.js";
await init();
canvas.width = innerWidth * devicePixelRatio; canvas.height = innerHeight * devicePixelRatio;
const bmp = await createImageBitmap(blob, { resizeWidth: canvas.width, resizeHeight: canvas.height });
const glass = await LiquidGlass.create(canvas, bmp, devicePixelRatio);

const bar = glass.addContainer(14, 0);                       // spacing (CSS px), z
glass.add(24, 24, 52, 52, -1, "regular", 0, true, false, bar, 0);   // x,y,w,h, radius(-1 capsule), material, tint, interactive, lens, container, z
glass.add(300, 24, 110, 52, -1, "tinted", 0x0088ff, true, false, -1, 0);
glass.setAppearance(0.5, false, false, false, matchMedia("(prefers-reduced-motion: reduce)").matches);

// Morphs and menus (handles are the numbers add/expandFrom return):
const menu = glass.expandFrom(0, 24, 90, 250, 300, 28, "regular", 0);  // source, x,y,w,h, radius, material, tint
glass.collapseInto(menu, 0);                                 // back into glass 0, then dropped
glass.setFrame(1, 300, 90, 110, 52); glass.remove(1);
glass.setContainerSpacing(bar, 20);                          // 0 = no merging
glass.setPhysics(false);                                     // changes land instantly
glass.setLightAngle(45);

let running = false, last = 0;
const tick = t => { const dt = (t - last) / 1000; last = t; running = glass.frame(dt); if (running) requestAnimationFrame(tick); };
const wake = () => { if (!running) { running = true; last = performance.now(); requestAnimationFrame(tick); } };
canvas.onpointerdown = e => { glass.pointerDown(e.clientX, e.clientY); wake(); };
canvas.onpointermove = e => { glass.pointerMove(e.clientX, e.clientY); wake(); };
canvas.onpointerup   = () => { glass.pointerUp(); wake(); };
wake();
```

The wrapper doesn't expose `set_material_with`, `place`, `glass_on_glass` or
`adapt_to_content` yet; add a method to `web/src/lib.rs` when a page needs one.

Canvas colour: canvases are `bgra8unorm`. Configure them with
`viewFormats: ["bgra8unorm-srgb"]` and render through the `-srgb` view. The shader outputs
linear light. The wasm wrapper already does this.

Map accessibility media queries:
- `prefers-reduced-transparency: reduce` → transparency 1 (or the reduce-transparency flag)
- `prefers-contrast: more` → increase contrast
- `prefers-reduced-motion: reduce` → reduce motion
- `prefers-color-scheme: dark` → dark

## 3. Path B: raw WebGPU with the shared WGSL

Use `assets/glass.wgsl` and `assets/pyramid.wgsl` unchanged (synced with liquid-rust 0.2.5).
Recreate the host from `liquid-rust/src/renderer.rs`.

**Pipelines** (`layout: "auto"`):
- `vs_glass` / `fs_glass`: triangle-strip, 4 vertices × N instances, premultiplied blending
  (`src: one, dst: one-minus-src-alpha`).
- `vs_full` / `fs_down_first`, `fs_down`: target `rgba16float`, no blend.
- `vs_full` / `fs_blit`: your target format.

**Bindings** (group 0): 0 `Globals` uniform, 1 `shapes`, 2 `groups`, 3 sampler, 4 backdrop
texture, 5 pyramid texture, 6 `materials` (all storage buffers read-only).

**Buffers** (std430, every field a vec4, all little-endian f32 except `info`):

```
Globals  (uniform, 64 B):   viewport[w,h,1/w,1/h]  misc[pxPerPt, pyramidLevels, scheme(1 light), 0]
                            light[sin a, -cos a, 0, 0]  touch[x px, y px, radius px, intensity]
Shape    (storage, 64 B):   center_half[cx,cy,hx,hy px]  params[radius px, shrink px, axisX, axisY]
                            stretch[along, 1/along, cornerExponent, 0]  extra[bevel depth px, 0, 0, 0]
Group    (storage, 48 B):   bounds[x0,y0,x1,y1 px]  info(u32)[firstShape, count, blend, 0]
                            geo[k px, presence, press, 0]
Material (storage, 144 B):  optics[refraction, adapt, dispersion, frostσ px]
  one per shape,            lighting[splay, specular, rim, rimShade]  edge[hairline, edgeBurn, contrast, dim]
  same index as `shapes`    shadow[offsetY px, σ px, opacity, 0]
                            light_a[lift, luminance, lumMix, lighten]  light_b[darkenTo, darken, 0, 0]
                            dark_a, dark_b (same)  tint[OKLab L, a, b, strength]
```

Each shape's material is resolved for that shape's own size. `blend` is 1 when a group's
members have different materials; the shader then mixes them across the liquid neck with the
smooth-min weights it uses for the outline. Write 0 when every member shares one material
(a row of equal buttons) so the shader skips the material loads. (Before 0.2.5 the group carried a 192 B thin/thick material pair and no
binding 6; update hosts ported from that layout.)

Effects are pre-multiplied by presence on the CPU, so materialize needs no alpha.

**Per frame**:
1. Blit content → target.
2. If the content changed, build the pyramid: mip i from mip i−1, scissored to the glass region.
   If it didn't, build it once for the whole screen and reuse it wherever the glass moves.
3. `drawIndexed`-free `draw(4, groupCount)` per z-layer.
4. Glass on glass: see `shader-math.md` §9.

**CPU side** (port it or call the wasm):
- springs: `motion.md`
- clustering with union-find on the container spacing
- material resolve per shape, which includes size thickness τ, the transparency slider and accessibility
- skip `queue.writeBuffer` when a buffer's bytes didn't change, and keep bind groups across
  frames (each one is a JavaScript object on the web)

## 4. Path C: three.js (r17x+, WebGPURenderer)

Render the scene to a `RenderTarget` first. Build the pyramid with mip-mapped
`RenderTarget`s or `texture.generateMipmaps`. Generated mips are box-filtered, so expect a
slightly blockier frost unless you add the 13-tap pass. Then draw a full-screen or instanced
quad whose `fragmentNode` implements `shader-math.md` §1–8 with TSL (`Fn`, `smoothstep`,
`texture(...).level()`). Keep the glass in an orthographic overlay scene.

## 5. Path D: CSS fallback

```css
.glass {
  /* Regular, light. Approximation: no bevel lensing, no liquid merge. */
  background: rgb(255 255 255 / .55);
  backdrop-filter: blur(8px) saturate(1.6);
  border-radius: 999px;
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / .9),      /* lit top rim   */
    inset 0 -1px 0 rgb(255 255 255 / .35),    /* opposite rim  */
    0 0 0 .5px rgb(0 0 0 / .18),              /* iOS 27 darkened edge */
    0 8px 15px rgb(0 0 0 / .06);
}
.glass .label { mix-blend-mode: plus-darker; color: #1a1a1a; }   /* kit: LINEAR_BURN labels  */
.glass.clear .label { mix-blend-mode: plus-lighter; color: #d9d9d9; } /* kit: LINEAR_DODGE glyphs */
@media (prefers-reduced-transparency: reduce) { .glass { background: rgb(245 245 245 / .94); } }
```

**Squircle corners in CSS:**
- `corner-shape` (CSS Borders 4) is available in Chrome/Edge 139+ and not yet in Safari or Firefox.
- `superellipse(K)` draws a corner with exponent 2^K inside the `border-radius` box, and the
  `squircle` keyword equals `superellipse(2)`, which is exponent **4**. That is *rounder-shouldered
  than iOS*.
- iOS (smoothing 60 % ≈ exponent 2.79 over 1.32 r) translates to
  `border-radius: calc(1.32 * R); corner-shape: superellipse(1.48);`
- Wrap it in `@supports (corner-shape: squircle)`. Elsewhere, fall back to `border-radius: R`
  or to an SVG `clip-path` built with a Figma-squircle path generator.

In Chromium only, `backdrop-filter: url(#lens)` with an SVG `feDisplacementMap` can add bevel
refraction. Precompute the displacement map from the squircle profile (kube.io's method).
Safari and Firefox ignore SVG backdrop filters, so treat this as progressive enhancement only.

## 6. Web-specific rules

- Size the canvas in device pixels. Pass `devicePixelRatio` as the scale; scene units are CSS pixels.
- Render on demand. Keep `requestAnimationFrame` running only while `frame()` reports motion,
  or while the backdrop is a playing video or a scrolling canvas.
- Pointer events go through `setPointerCapture`, and the canvas needs `touch-action: none`
  so drags don't scroll the page.
- Keep text in the DOM above the canvas for accessibility and selection. Give it vibrancy with
  `mix-blend-mode`.
- One canvas, one device. Don't create a WebGPU context per glass element.
