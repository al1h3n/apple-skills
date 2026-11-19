# App icons: layer model, glass finish, generator recipe

Current look = 27. The 26 look = `versions/26.md` §4.

## 1. Model [Apple, HIG App icons + Icon Composer]

Icon = **background layer + 1…n foreground layers**, grouped along Z. Max **4 groups**. System adds the glass; artwork stays flat.

| Platform | Canvas | Mask (applied by system) |
|---|---|---|
| iOS, iPadOS, macOS | 1024 × 1024 | rounded rectangle (continuous) |
| watchOS | 1088 × 1088 | circle |
| visionOS | 1024 × 1024 | circle |
| tvOS | 800 × 480 | rounded rectangle |

Colour: sRGB, Display P3, Gray Gamma 2.2.

Per layer: colour, opacity, blend mode, fill, position, visibility, **glass on/off** (off = flat vector, for pre-rendered art).

Per group:

| Property | Options |
|---|---|
| Specular | Automatic, Inside (foreground darker than background), Outside (reverse). Sharp, with dark parts defining the shape |
| Blur | frosts the layers below |
| Refraction | height (≈ bevel corner radius) + strength (how far artwork is pulled in); inverse exists |
| Translucency | clear toward the bottom, colour kept at the top. Keep low |
| Shadow | Neutral (desaturated) or Chromatic (takes the layer's colour). "Ring" shadow: tight, edge-defining |
| Blend | normal; darken, multiply, plus darker; lighten, screen, plus lighter |

Background: solid, gradient, or automatic (system light / system dark). Can be generated from one brand colour.

Appearances: **default, dark, clear light, clear dark, tinted light, tinted dark**. Same features in all; dark more subdued; clear and tinted more so. System generates missing ones. One **mono** greyscale version → all clear + tinted variants: deep darks, bright highlights, part of foreground near white. **Clear-mode glass is pinned**: ignores the transparency slider.

Bitmap (PNG) icon: system segments it into foreground + background itself; glass goes on "large, simple shapes", not on text or thin lines.

## 2. Artwork rules [Apple]

- Clear, hard edges on foreground layers. No feathering.
- **Bake nothing in**: no highlights, drop shadows, bevels, blurs, glows.
- Square, unmasked layers. Pre-masked → jagged edges, broken specular.
- Filled, overlapping shapes over outlines. Text only if essential. No photos, no thin lines, no sharp corners.
- Vary foreground opacity for depth.
- To pass one element behind another: split into layers.
- Test at the smallest size, next to other icons. Never rely on colour alone.

## 3. What the glass finish is [refs, M2]

Five small things per foreground group. Nothing else: no gloss streaks, no reflections, no noise.

| # | Effect | Seen on |
|---|---|---|
| 1 | **Translucency ramp**: opaque where light hits, lets ≈ 15–20 % of the background through at the far side | Roblox slab: near-white top-left, bluish bottom-right |
| 2 | **Edge pair**: thin bright line on the lit edges, faint darker line on the opposite edges | every shape edge |
| 3 | **Tight shadow in the layer's own colour** (chromatic), ≈ 3 % of icon size, offset down | Roblox: deeper blue, not grey. Chrome: each segment's colour bleeds onto the white, 2–3 % |
| 4 | Slight blur of what sits under a translucent part | petal overlaps |
| 5 | Clean separation between groups: overlaps are crisp lens shapes in a clean mixed colour | Photos petals |

Plus: background gets a soft vertical gradient (lighter top) and a thin lighter line just inside the tile edge. Holes show the background with a light line on the far inner edge.

Highlights sit at top and bottom, **fixed**: they do not follow device tilt.

Clear style on the Home Screen: tile = clear glass, frost ≈ 2 (σ 0.8 pt), refraction band 3–4 pt on a 62 pt icon, bright rim tinted by the wallpaper; glyph ≈ 85–90 % opaque white [M2].

Same move elsewhere: status-bar battery = one solid filled capsule whose fill is the level. Filled shapes over outlines.

## 4. Generator recipe [D, tuned against the references]

Input: background + 1–4 foreground groups, bottom → top. `S` = icon size (1024). Light from
the top, a little from the left (direction (−0.3, −1)). Per group, in order (27 values; tuned
against refs 16 and 18 with LIGMA, see below):

```
1  coverage A (layer alpha); signed distance + normal from A (a mask field)
2  shadow    : colour = chromatic ? mix(layer colour, black, 0.35) : black
               offset (+0.004 S, +0.012 S), blur σ 0.018 S, opacity 0.30   [26: (0, 0.02 S), σ 0.045 S, 0.22]
3  glass     : bend and blur what is under A: mask glass, refraction 0.6, bevel 0.035 S,
               profile n = 2, blur σ 0.006 S, no face, no light, no shadow of its own
               [26: refraction 0.4, bevel 0.06 S, n = 3, σ 0.02 S]
4  fill      : layer colour, opacity = 1 − translucency · (0.3 + 0.7 t)
               t = 0 at the lit edge → 1 at the far edge, along the light; translucency 0.25
               [26: 0.45, plus white mixed in toward the far edge (×0.25 t): the milky slab]
5  light     : light line round every edge: width 0.005 S, strength 0.8 × (0.35 + 0.65 facing^1.5)
               dark line on the far edges: width 0.003 S, strength 0.15 × away^1.5
               [26: width 0.02 S, strength 0.4, no dark line]
then         : tile edge light (width 0.006 S, 0.3), platform mask
```

Translucency fades from 30 % of its value at the lit edge, not from zero: Apple's slabs let a
little through everywhere. The light line runs round every edge at a third of its strength,
since Apple lights glass from two opposite sides.

Background: solid, or two-stop vertical gradient from one colour (top ≈ 8 % lighter). Mask last.

Appearances:

| Appearance | Background | Foreground |
|---|---|---|
| Default | as authored | as authored |
| Dark | near-black gradient | hue kept, luminance lifted |
| Clear light / dark | clear glass tile (pinned) | mono luminance on glass |
| Tinted light / dark | dark / light plate | mono luminance × tint |

Mono source = artwork luminance stretched to full range.

Flat single PNG in → split first: background = dominant border colour; foreground = connected colour regions, merged down to ≤ 4 groups by area and colour distance.

Report pixel difference against a reference; never claim a match.

**LIGMA** (`github.com/al1h3n/LIGMA`) implements this recipe: `ligma-core` (Rust, liquid-rust
mask glass for step 3), a CLI (`ligma split | render | export | compare`) and a Tauri 2 app
(Svelte 5, Bits UI, the glass kit). Against Apple's glass versions it measures 3.5 % mean
difference for Roblox (ref 15 → 16) and 6.6 % for Chrome (19 → 18); Apple's Chrome is far
more saturated than its flat logo, which no recipe value explains. Full-bleed flats render at
scale 1; logos on a transparent edge get a white tile at scale 0.8.

### macOS 27 look [M9, 71 icons]

Against 26 the median icon barely changes colour, but where it changes it is purer
(saturation × 1.1) and almost every icon is sharper (edge sharpness × 1.43, 62 of 71):
glyphs on colour are near-solid (translucency ≈ 0.1; iOS 27 keeps ≈ 0.25), layers overlap
crisply with nothing blurred under them (blur ≈ 0.0015 S), every layer and the tile have a
thin light line on the upper edge, light tiles are lighter and cleaner, inner shadows are a
tight ring. LIGMA renders this for `--platform macos --look 27`.

## 5. Why [principles]

Material **in** the artwork at the edges, not **over** it. Recognition first, material as finish. Effect kept where it carries information (edge, depth order).
