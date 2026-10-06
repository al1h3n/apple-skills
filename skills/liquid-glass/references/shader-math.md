# Shader Maths for Apple-grade Liquid Glass

This is the algorithm used by `assets/glass.wgsl`, the shader that powers Liquid Rust. The
numbers are calibrated by comparing pixel profiles against Apple's Figma kit render. Read
this before writing glass in WGSL, GLSL, HLSL, Metal or Unity. The WGSL in `assets/` is the
tested reference, so port it rather than retyping it.

## 0. Pipeline overview

```
content texture ──► pyramid (13-tap downsample ×N, Rgba16Float, region-limited, cached)
                         │
per glass GROUP (one instanced quad, bounds = shapes + shadow margin):
   SDF of all shapes (smooth union, k = container spacing) + analytic gradient
   outside  : drop shadow (offset SDF, erf falloff, luma-adaptive) + burn hairline
   inside   : refract through squircle bevel (Snell, 3 IORs = dispersion)
              → frost (pyramid, B-spline, continuous level)
              → tone (Figma blend fills, sRGB-encoded space; light/dark faces)
              → tint (OKLab, lightness follows backdrop)
              → lit rim line, inner rim shadows (offset-SDF), touch glow
   output premultiplied: rgb·coverage, alpha = coverage + (1-coverage)·darkening
```

Everything works in **pixels**. Convert points to pixels with the device pixel ratio, and keep all kit values in points.

## 1. Signed distance and normal

Use a rounded box with an analytic gradient. Inside the straight section, soften the normal
across the diagonal so the bevel doesn't crease at the corners. This does the same job as
Core Animation's `gradientOvalization`:

```wgsl
fn sdg_rrect(p: vec2f, half: vec2f, r: f32, soft: f32) -> vec3f {   // (d, normal)
    let sg = select(vec2f(-1.0), vec2f(1.0), p >= vec2f(0.0));
    let w = abs(p) - (half - vec2f(r));
    let g = max(w.x, w.y);
    if g > 0.0 { let q = max(w, vec2f(0.0)); let l = length(q); return vec3f(l - r, sg * q / max(l, 1e-6)); }
    let e = exp((w - vec2f(g)) / max(soft, 1e-3));      // soft = 0.35 × bevel depth
    return vec3f(g - r, sg * normalize(e));
}
```

Capsule: `r = min(half)`.

**Squircle (continuous) corners.** Model the corner as a superellipse. Divide the norm by its
gradient length so it is a true distance at the edge. Replace the `g > 0` branch with:

```wgsl
let q = max(w, vec2f(1e-7));                       // r = corner extent, n = exponent
let l = pow(pow(q.x, n) + pow(q.y, n), 1.0 / n);
let grad = pow(q / l, vec2f(n - 1.0));
let gl = length(grad);
return vec3f((l - r) / gl, sg * grad / gl);
```

Map Figma or iOS smoothing `s` onto it. This is fitted to Figma's exact arc-plus-two-Bézier
geometry, with a maximum error of 0.74 % of the radius for any `s`:

```
budget = min(half);  r = min(radius, budget)
s      = min(s, budget / r - 1)            // Figma clips smoothing that doesn't fit
extent = r · (1 + 0.42 s + 0.18 s²)        // iOS (s = 0.6): 1.317 r
n      = 2 + 0.95 s + 0.6 s²               // iOS: 2.79   (s = 0 → circle, n = 2)
```

The interior normal softening (`soft`) works unchanged. For one `pow` pair per corner pixel
you get Apple's corner, so there's no need for exact Bézier distance.

To stretch a lens being dragged, rotate into the velocity frame,
divide the coordinates by `(1+s, 1/(1+s))`, and multiply the distance by `min(scale)`.

## 2. Liquid merge: smooth union with the gradient (Quilez)

```wgsl
fn smin_g(a: vec3f, b: vec3f, k: f32) -> vec3f {
    if k <= 0.0 { return select(b, a, a.x < b.x); }
    let h = max(k - abs(a.x - b.x), 0.0);
    let m = 0.25 * h * h / k;
    let n = 0.5 * h / k;
    return vec3f(min(a.x, b.x) - m, mix(a.yz, b.yz, select(1.0 - n, n, a.x < b.x)));
}
```

- `k` is the container spacing in pixels. Shapes closer than `k` grow a neck and fuse.
- On the CPU, group shapes whose bounding-box gap is less than `k` (union-find). Draw one quad per
  group and loop only over that group's shapes. Never loop over every shape in a full-screen pass.
- Antialiasing: `coverage = clamp(0.5 - d/|∇d|, 0, 1)`. The union's gradient is shorter than 1
  inside the neck.
- A shape that materializes next to a neighbour should grow out of it: add
  `(1 - presence) × min(half)` to its distance. An isolated shape scales in by only 15 %.

## 3. Refraction through the bevel

Bevel width = **depth** (Figma *Depth*, in points), clamped to the shape's half-size.
`t = clamp(-d / depth, 0, 1)` runs from 0 at the rim to 1 where the flat top starts.

Use Apple's squircle profile `h(t) = (1 - (1-t)^4)^(1/4)`, whose slope is

```
h'(t) = (1-t)^3 · (1 - (1-t)^4)^(-3/4)
```

Treat it as a vertical ray hitting a surface tilted by θ = atan(h'), refracted with η = 1.5,
travelling one glass thickness:

```wgsl
fn refract_offset(slope: f32, eta: f32) -> f32 {
    let s = slope / sqrt(1.0 + slope * slope);
    return tan(asin(s) - asin(s / eta));          // 1.117 at the rim for η = 1.5
}
offset = refraction × depth × refract_offset(h'(t), η)       // Figma Refraction 0..1
sample = p - n × offset                                      // pull from further inside
```

- **Dispersion**: use three offsets with η ∓ 0.25·dispersion and read R, G and B from separate
  samples. Do this only inside the bevel. The flat centre takes one sample and stays colour-clean.
- The rim pulls content from roughly 1.1·depth·refraction inside. That compression band is
  the main "Liquid Glass" cue.

## 4. Frost from one pyramid

Build the pyramid once per frame:
- Level *n* (n = 1…7) is a 2× downsample with Jimenez's 13-tap filter. The weights are centre 0.125,
  the four diagonal ±1 taps 0.125, the four axis ±2 taps 0.0625 and the four corner taps 0.03125.
- Store E[luminance²] in the first level's alpha. It averages down the chain, so every level
  carries the local variance used by the adaptive shadow.
- Only render inside the region that glass can sample: the group bounds plus 1.2·depth,
  8σ and 8 px. Cache the pyramid while the content hasn't changed.

Sampling for a frost blur σ (σ = Figma Frost / 2, in pixels):

```
level = log2(σ / 1.9) + 1             // level-1 σ ≈ 1.9 px: kernel+bilinear var 1.75, B-spline var 1.33
level ≤ 0  → sharp source
0 < level < 1 → mix(source, bspline(mip 0), level)
else          → mix(bspline(floor), bspline(floor+1), fract)     // continuous blur radius
```

Get the 1.9 wrong and the frost comes out twice as soft. That mistake makes a result look
"2020 glassmorphism". B-spline fetch: 4 bilinear taps, with the weights from the cubic B-spline basis.

## 5. Tone mapping (the "Fill + Shadow" layer)

Do this in **sRGB-encoded** space, as Figma does: `encode(x) = x^(1/2.2)`.

```
x = min(c + lift, 1)                          // LINEAR_DODGE grey
x = mix(x, SetLum(x, target), lum_mix)        // LUMINOSITY (W3C SetLum + ClipColor, lum = .3/.59/.11)
x = mix(x, 1, lighten)                        // LIGHTEN with white
x = mix(x, min(x, darken_to), darken)         // DARKEN with grey
```

| Preset | lift | luminance / mix | lighten | darken |
|---|---|---|---|---|
| Clear (kit) | 0.063 | 1.0 / 0.04 | 0 | 0 |
| Clear bar (Dock) | 0 | 0.6 / 0.33 | 0 | 0 |
| Regular light, small → large | 0 | 0.92 / 0.3 | 0.2 → 0.7 | 0.75 / 0.1 |
| Regular dark (derived) | 0.03 | 0.25 / 0.2 | 0 | 0 / 0.25 → 0.52 |

Regular glass mixes the light and dark faces using `lightness`:
`mix(scheme, smoothstep(0.26, 0.46, L_backdrop), adapt)`. Here `L_backdrop` is the coarsest
pyramid level at the group centre, and `adapt = 1 - τ` with `τ = smoothstep(44, 320, min side in pt)`.
Small glass flips with the content; large glass follows the colour scheme. The threshold is
biased toward light.

## 6. Tint (stained glass)

Work in OKLab:

```
L = clamp(L_tint + (L_backdrop - 0.65) × 0.6, 0.15, 0.97)
tinted = OKLab(L, a_tint, b_tint)
g = mix(g, tinted, 0.85 × alpha)      // never 1.0: a solid fill "breaks the material"
```

## 7. Light

1. **Lit rim line**, from Figma *Light* and *Splay*. It's crisp: width about 0.25 pt × (1 + splay).
   ```
   facing = dot(n, to_light)                         // to_light = (sin a, -cos a), y down
   dir    = facing > 0 ? splay + (1-splay)·facing²  :  splay + (0.4-splay)·facing²
   line   = exp(d / (0.25pt·(1+splay)))               // d ≤ 0 inside
   hl     = vivid(g)·1.45 + 0.05                      // saturation ×2.2, like CA's vibrantColorMatrix
   g      = mix(g, hl, clamp(line·dir·light·2.5, 0, 1))
   ```
   In the kit, the top rim is about 254/255 over a ~175 face, the bottom about 205, and the sides 190–212.
2. **Inner shadows** (the kit's "Glass Effect" layer). Each one is a Figma inner shadow evaluated
   exactly with an offset SDF:
   `shadow(p) = Φ((sd(p - offset) - grow) / σ)`, where σ = blur / 2 and grow = -spread.
   - Rims: offset ±40 pt along the light axis, grow 40 pt, σ 5 pt, **+0.157** (`#282828` dodge) on both edges.
   - Shade: offset toward the light side, grow 40, σ 15 pt, **−0.098** (`#E6E6E6` burn). Clear glass only.
   - Hairlines (dock): offset ±1.25 pt, grow 0, σ 0.125 pt, +0.157.
   - Skip these when `-d > 45 pt`; past that depth they are provably below Φ(-3).
3. **Touch glow**: `g += 0.22 · I · exp(-|p - touch|² / r²)`, using one scene-wide light so the
   glow spills onto neighbouring glass.

## 8. Outside the shape

- **Drop shadow**: evaluate the SDF at `p - (0, offset_y)` and take `1 - Φ(d/σ)`. The kit uses
  (0, 8 pt), blur 15, 2 % for controls, up to blur 48 and 25 % for big menus. Make it
  luma-adaptive: `×mix(0.7, 1.6, smoothstep(0.02, 0.12, stddev))` over busy content and
  `×mix(1, 0.65, smoothstep(0.5, 0.85, mean))` over light content.
- **Darkened edge** (iOS 27): burn 0.2 over a 0.5 pt ring outside the shape, plus 0.184 on the
  left and right flanks (shape shifted ±1.25 pt and shrunk 0.75 pt). Express the burn as
  multiplicative darkening, `1 - L(burnt)/L(under)`, so it composes as premultiplied alpha.

## 9. Glass on glass

Sort groups into z-layers. If layer *i* overlaps the bounds of lower glass:
1. copy the current backdrop to an offscreen texture inside layer *i*'s region;
2. draw layer *i−1* into it;
3. rebuild the pyramid there;
4. draw layer *i* from it.

Use two textures in ping-pong. Layers that don't overlap reuse the pyramid. Upper glass
should be Regular.

## 10. Cost guide (1206×2622, RTX 30-series)

| Item | Cost |
|---|---|
| Pyramid, region-limited | ~0.1 ms, 0 when cached |
| Flat interior pixel | 1 SDF + 8 bilinear taps |
| Bevel pixel with dispersion | 1 SDF + 24 taps + 2–5 offset SDFs |
| Whole showcase (8 groups, merging, layered) | 0.6–0.7 ms per frame |

Mobile guidance:
- Drop dispersion, which takes the bevel from 24 taps to 8.
- Cap the pyramid at 5 levels.
- Scale the inner-shadow σ by DPR rather than adding samples.

## 11. Common bugs

| Symptom | Cause |
|---|---|
| Looks like frosted plastic | blur too strong (σ calibration) or a white fill instead of luminance compression |
| Hard crease along corner diagonals | raw box gradient: soften the interior normal |
| Seams between merged blobs | hard `min`, or normals not mixed in `smin` |
| Jagged edge | coverage not divided by \|∇d\| |
| Rim looks like a 1 px white border | missing direction factor / splay |
| Colour fringe in the centre | dispersion not restricted to the bevel |
| Washed-out results on the web | writing linear colour to a `bgra8unorm` canvas: render through a `-srgb` view |
| Glass flickers light/dark | sampled a fine level for adaptivity: use the coarsest level at the group centre |
