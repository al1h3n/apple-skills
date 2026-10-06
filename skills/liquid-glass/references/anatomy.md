# Anatomy of Apple's Liquid Glass

What the material actually is, layer by layer, with the exact numbers Apple ships in its
own design kit and in its compositor. Use this file whenever you need to *reproduce* the
look (shaders, Rust engine, Figma) or *judge* whether an implementation looks right.

Confidence tags used below:
- **[Apple]** stated by Apple (WWDC session, HIG, developer docs).
- **[Kit]** measured from Apple's official iOS 26/27 Figma UI kit ("Template for UI" →
  frame "iOS + IpadOS"; effect stacks read through the Figma Plugin API).
- **[RE]** reverse-engineered from the private Core Animation pipeline (ShatteredGlass).
- **[Derived]** our own inference; tune by eye.

---

## 1. What makes it *Liquid Glass* and not glassmorphism

| Glassmorphism (2020) | Liquid Glass (2025+) |
|---|---|
| Big Gaussian blur, white fill at 10–30 % | Small blur ("frost" 3–16 pt). The material **bends** light instead of scattering it [Apple: "previous materials scattered light, this new set of materials dynamically bends, shapes, and concentrates light"] |
| Flat interior | Curved **bevel**: a band along the edge where the backdrop is refracted (pulled from further inside) and slightly chromatically split |
| 1 px white border | Directional **specular rim** (light from the top, weaker on the opposite edge) + thin bright rims + a **darkened outer hairline** (iOS 27) |
| Fixed light or dark tint | **Luminance compression**: the backdrop's dynamic range is squeezed toward a mid/light/dark tone, hue and saturation stay. Small elements flip light↔dark with the content behind them [Apple] |
| Fade in / fade out | **Materializes** by ramping the lensing up from zero [Apple] |
| Static | Responds to touch: flexes, scales, **illuminates from inside starting under the finger**, the glow spills onto nearby glass [Apple] |
| Separate cards | Nearby elements **merge like liquid** (SDF smooth union) and morph into each other [Apple: GlassEffectContainer] |

If an implementation is missing the bevel refraction, the dual rim, the luminance
compression or the merge behavior, it will read as "frosted glass", not "Liquid Glass".

## 2. The layer stack (Apple's Figma kit) [Kit]

Every glass control in the kit is a component with two sibling frames of identical shape:

```
BG (component)
├── Fill + Shadow   ← tone mapping fills + outer shadows/hairlines   (below)
└── Glass Effect    ← GLASS effect + inner shadows (rim light)        (above)
<content>           ← glyphs/labels with vibrancy blend modes         (on top)
```

The glass layer refracts what is *under* it, i.e. the backdrop **after** the fill layer's
tone mapping. Order of operations per pixel:

1. backdrop → tone-map (fills of "Fill + Shadow")
2. → refract + frost + disperse + specular (GLASS effect)
3. → inner shadows (rims), composited with blend modes
4. outside the shape: drop shadow + darkened hairlines from "Fill + Shadow"
5. content on top with vibrancy

### 2.1 Clear glass — "Clear Glass" (Lock Screen buttons, Control Center tiles)

```
Fill + Shadow
  fills    #101010  LINEAR_DODGE 100 %        (+0.063 lift)
           #FFFFFF  LUMINOSITY    4 %         (luminance 4 % toward white)
  effects  DROP_SHADOW (0, 8)   blur 15  spread 0     #000000  2 %   NORMAL
           DROP_SHADOW (0, 0)   blur 0   spread 0.5   #CCCCCC 100 %  LINEAR_BURN   ← 0.5 pt dark hairline
           DROP_SHADOW (-1.25,0) blur 0  spread -0.75 #D0D0D0 100 %  LINEAR_BURN   ← extra hairline, left
           DROP_SHADOW (+1.25,0) blur 0  spread -0.75 #D0D0D0 100 %  LINEAR_BURN   ← extra hairline, right
Glass Effect
  GLASS    light 0° @ 40 %, refraction 70, depth 30, dispersion 20, frost 6, splay 20
  INNER_SHADOW (0, +40) blur 30 spread -40 #E6E6E6 LINEAR_BURN   ← broad soft shade under the top edge
  INNER_SHADOW (0, -40) blur 10 spread -40 #282828 LINEAR_DODGE  ← bright rim, bottom edge
  INNER_SHADOW (0, +40) blur 10 spread -40 #282828 LINEAR_DODGE  ← bright rim, top edge
```

These are exactly the values in the user-supplied Figma panel screenshots.

### 2.2 Clear glass, chrome variant — Dock and Home-Screen search pill ("Materials")

```
Fill + Shadow  fill #999999 LUMINOSITY 33 %   (luminance pulled 33 % toward 0.6 → legibility)
               shadows/hairlines identical to 2.1
Glass Effect   GLASS light 0° @ 20 %, refraction 70, depth 30, dispersion 20, frost 3, splay 20
               INNER_SHADOW (0,-40) blur 10 spread -40 #282828 LINEAR_DODGE
               INNER_SHADOW (0,+40) blur 10 spread -40 #282828 LINEAR_DODGE
               INNER_SHADOW (0,-1.25) blur 0.25 spread 0 #282828 LINEAR_DODGE  ← crisp 1.25 pt top hairline
               INNER_SHADOW (0,+1.25) blur 0.25 spread 0 #282828 LINEAR_DODGE  ← crisp bottom hairline
corner 38 pt on a 103 pt dock (continuous corner)
```

### 2.3 Regular glass, large & light — Home-Screen Quick Actions menu

```
Fill + Shadow  fills #FFFFFF LIGHTEN 70 %, #BFBFBF DARKEN 10 %
               DROP_SHADOW (0, 8) blur 48 #000000 25 %                ← "deeper, richer shadow" of big glass
               hairlines as 2.1 but colour #DBDBDB (lighter burn, -0.14)
Glass Effect   GLASS light 0° @ 25 %, refraction 70, depth 30, dispersion 20, frost 16, splay 20
               rims: the two #282828 LINEAR_DODGE inner shadows (blur 10); top shade disabled
content        labels #1A1A1A with layer blend LINEAR_BURN (vibrant dark text)
               destructive row #FF383C solid (iOS 26 systemRed), no blend
corner 30 pt
```

### 2.4 Lock-Screen clock (glass text)

`GLASS refraction 50, depth 1, frost 0, light 40 %, dispersion 0, splay 20` on the glyph
outlines + inner shadows (0,±1) blur 3 #282828 LINEAR_DODGE, over a fill copy
`#404040 LINEAR_DODGE` with background blur 80. Depth 1 = a 1 pt bevel: only the outline
lenses. This is how "Liquid Glass text" is built.

### 2.5 What sits *on* glass [Kit]

| Element on a glass tile | Construction |
|---|---|
| Glyph on clear glass | `#D9D9D9` LINEAR_DODGE, layer opacity 90 % (vibrant white) |
| Secondary text | white 33 % LINEAR_DODGE |
| Circular sub-button (AirDrop, Focus icon) | **not glass**: plain circle filled `#111111 LUMINOSITY 60 %` + `#777777 COLOR_DODGE` + `#222222 LINEAR_DODGE` |
| Active toggle (Wi-Fi, Bluetooth) | **not glass**: opaque `#0088FF` (iOS 26 systemBlue) / `#34C759` circle, white glyph |
| Slider fill | `#6F6F6F COLOR_DODGE` + white 65 % SCREEN, coloured glyph on top (sun `#FFCC00`, speaker `#00C0E8`) |
| Lock-screen widget text | white / `#999999` LINEAR_DODGE (vibrancy = additive "plus-lighter") |

Takeaway: **glass is used exactly once per stack.** Everything above it is a fill with a
vibrancy blend mode, or a solid semantic colour.

### 2.6 Figma GLASS parameters, decoded

| Figma | Plugin API | Meaning | Kit range |
|---|---|---|---|
| Light angle | `lightAngle` (deg) | direction the light comes from; 0° = top | 0 |
| Light % | `lightIntensity` 0–1 | specular rim strength | 0.2–0.4 |
| Refraction | `refraction` 0–1 | strength of bevel displacement | 0.5–0.7 |
| Depth | `depth` ≥ 1 | **bevel width in pt**: how far the curved edge reaches inward. 30 on a 58 pt circle = whole dome | 1–30 |
| Dispersion | `dispersion` 0–1 | chromatic split inside the bevel | 0–0.2 |
| Frost | `radius` | backdrop blur radius (≈ 2σ) | 0–16 |
| Splay | `splay` 0–1 | how widely the light spreads around the rim | 0.2 |

### 2.7 Corners: squircles everywhere [Kit] [Apple]

Apple's corners are **continuous-curvature** ("squircle"): the curve eases out of the
straight edge instead of jumping onto a circle. In Figma this is **corner smoothing 60 %**.
Every corner radius in the kit's "iOS + IpadOS" frame was read:

| Element | Size (pt) | Radius | Smoothing | Effective shape |
|---|---|---|---|---|
| Lock Screen buttons, CC 1×1 tiles, AirDrop/Wi-Fi/Focus circles | 58–70 | ≥ half | 0.6 | circle (smoothing has no room) |
| CC 2×1 Focus, Volume, Brightness sliders | 155×70, 70×155 | 34–100 | 0.6 | capsule / near-capsule |
| CC 2×2 tiles (Music, Group, Floor Lamp) | 142–155 | 30–34 | 0.6 | **squircle** |
| Quick Actions menu | 250×350 | 30 (outer instance 12) | 0.6 | **squircle** |
| Dock, dock material | 368×103, 740×107 | 38 | 0.6 | squircle, smoothing clipped by the height |
| Search pill | 77×30 | 38 | 0.6 | capsule |
| Folder tiles in the dock stack, album cover | 52–68 | 12–17 | 0.6 | **squircle** |
| Widget bar, page control | 5×54, 48×7 | 3–50 | 0.6 | squircle / capsule |
| Dynamic Island, Home indicator, badges, dots, menu-bar items, the clock widget | — | — | **0** | circular/capsule (hardware-matched or tiny) |
| App icons | — | — | — | image masks: Apple's own icon squircle (`smooth 0` in the kit, because the mask lives in the bitmap) |

How smoothing behaves (Figma algorithm, also iOS):
- A corner with radius `r` and smoothing `s` reaches `(1 + s)·r` along each edge.
- Where that doesn't fit in half the shorter side, the smoothing shrinks. So circles and
  capsules stay round, and only "real" rounded rectangles become squircles.
- At 45° a smoothed corner passes almost exactly where the plain arc does. The difference is
  in how gently it leaves the straight edge: no curvature jump, so no visible "kink" in the
  highlight and no crease in the refraction band.

Rule: any glass rounded rectangle or panel should use continuous corners. SwiftUI's
`.rect(cornerRadius:)` and `RoundedRectangle` already default to `.continuous`. Liquid
Rust's `Shape::Rounded` is continuous. Use circular corners only to match hardware-like
elements (Dynamic Island) or when you need CSS parity.

## 3. The compositor pipeline (Core Animation, macOS 26) [RE]

SwiftUI's glass is a `CABackdropLayer` with a private `glassBackground` CAFilter fed by a
signed-distance-field layer (`CASDFLayer` + `CASDFElementLayer`, union op, smoothness 8),
plus two `CASDFGlassHighlightEffect` layers. Default values captured from a capsule control:

| Group | Parameter | Value | Reading |
|---|---|---|---|
| Backdrop | `scale` | **0.25** | the backdrop is sampled at quarter resolution — blur and refraction never need full-res |
| | `tracksLuma`, `allowsFilteredLuma` | true | adaptivity (light/dark, shadow strength) reads backdrop luminance |
| Blur | `inputBlurRadius` | 1.5 (at 0.25 scale ≈ 6 pt) | frost |
| | `inputBlurDistance0…4` / `Opacity0…4` | −25,−1,0,0,10 / 1,.5,.5,1,1 | blur profile varies with SDF distance |
| Refraction | `inputInnerRefractionAmount` / `Height` | −40 / 12.5 | strong inward pull inside a 12.5 pt band |
| | `inputOuterRefractionAmount` / `Height` | 10 / 6.25 | slight lensing *outside* the edge |
| Face | `inputFaceColorMatrixBlack` / `White` / `Saturation` | 0.1 / 0.3 / 1 | dynamic-range compression of the backdrop |
| | `inputFaceColorMatrixFillColor` | white α 0.02 | the colour hook used by `.tint()` |
| Shadow | offset / blur / opacity | (0,8) / 40 / 0.4, `inputShadowVibrancyContribution` 0.01 | luma-aware drop shadow |
| Bleed | amount / blur / opacity | 17.5 / 35 / 0.01 | light from content bleeding into the shadow |
| HDR | `inputMaxHeadroom` | 9999 | highlights are allowed into EDR headroom |
| Highlight | `CASDFGlassHighlightEffect` ×2 | angles +45° and −135°, amount 0.5, curvature 1.7, spread 1.59, height 1 | **two opposite rim lights** |
| Highlight tone | `vibrantColorMatrix` | saturation ≈ 2.8, gain 1.45, +0.05 | the rim is a brightened, *saturated* copy of the backdrop, not flat white |

## 4. Adaptive behaviours to reproduce [Apple, WWDC25 "Meet Liquid Glass"]

1. **Lensing** defines the shape; blur is secondary.
2. **Size → thickness.** Larger glass (menus, sidebars) is "thicker": deeper shadow, more
   pronounced lensing, softer scattering, more opaque face. Small glass is thin and clear.
3. **Shadow adapts to content**: more opaque over text/busy content, lighter over flat light
   backgrounds.
4. **Tint and dynamic range shift** continuously so glyphs stay legible.
5. **Small elements flip light/dark** (nav bars, tab bars); large ones (menus, sidebars) don't.
   Glyphs on regular glass flip with it.
6. **Highlights** come from light sources in an environment; on lock/unlock they travel
   around the silhouette; some respond to device motion.
7. **Interaction**: "instantly flexing and energizing with light", gel-like flexibility,
   glow starts under the fingertip and spreads to nearby glass.
8. **Materialize**: appear/disappear by modulating light bending, never by alpha fade.
9. **Morph**: shapes in one container blend as they approach (spacing threshold) and morph
   between states (matched geometry).
10. **Window focus**: inactive windows recede (iPad/Mac) — dim glyphs, flatten glass.

### iOS 27 / macOS 27 deltas [Apple, WWDC26 keynote + Platforms State of the Union]

- Material "more effectively diffuses complex content behind it" → slightly more frost and
  stronger luminance compression than iOS 26.
- Darkened outer edge + brighter specular → the burn hairlines in §2 are now default.
- **Transparency slider** in Settings: *ultra clear → fully tinted*, replacing the iOS 26.1
  binary *Clear / Tinted* switch. Apps get it automatically.
- Sidebars extend to the window edges on iPad/Mac and keep refracting the wallpaper.
- Icons: sharper, multi-layer glass, optional per-layer refraction (Icon Composer).
- Xcode 27: rebuilding adopts Liquid Glass; the `UIDesignRequiresCompatibility` opt-out is
  on its way out — plan for glass, don't fight it.

## 5. Checklist: does it look like Apple?

- [ ] Backdrop visibly *bends* within ~10–30 pt of the edge, flat in the centre.
- [ ] Bright rim at top **and** bottom (light from top, weaker opposite), not a uniform border.
- [ ] A hair-thin darker line just outside the edge (iOS 27).
- [ ] Interior is the backdrop with compressed contrast, *not* a white or grey overlay.
- [ ] Faint rainbow only inside the bevel; centre is colour-clean.
- [ ] Neighbouring shapes in one group pull toward each other and fuse before touching.
- [ ] Press: grows a little, lights from the finger, settles with a small bounce.
- [ ] Appear: lensing ramps in; no opacity crossfade.
- [ ] 120 Hz smooth, frame-rate independent; idles at 0 fps when nothing moves.
