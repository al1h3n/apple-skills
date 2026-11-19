# Anatomy: what Liquid Glass is made of

Use to reproduce (shader, engine, Figma) or to judge a result. Numbers = current OS (27). Older: `versions/26.md`. Per-component sizes: `components.md`. Tags: `principles.md` head.

Kit = Apple's Figma libraries "iOS and iPadOS 27" (198 components) + "macOS 27" (115), every variant read through the plugin API.

## 1. Not glassmorphism

| Glassmorphism | Liquid Glass |
|---|---|
| Big blur, white fill 10–30 % | Small frost (3–16). Material **bends** light [Apple] |
| Flat interior | **Bevel**: band along edge where backdrop is refracted + faintly split in colour |
| 1 px white border | **Rim light** top + bottom, **dark burn hairline** outside [Kit] |
| Fixed tint | Tone layer below the lens; small glass flips light/dark with backdrop [Apple][M8] |
| Fade in/out | **Materializes**: lensing ramps from zero [Apple] |
| Static | Press: grows a little, lights from the finger [Apple][M3] |
| Separate cards | Neighbours in one group **merge** (SDF smooth union) [Apple] |

Missing bevel, rim pair, tone layer or merge → reads as frosted plastic.

## 2. Layer stack [Kit]

Same sandwich everywhere, bottom → top:

```
Fill + Shadow    tint paints + drop shadow + 3 zero-blur "shadows" = outer hairline      (under the lens)
White Backing    tinted glass only: #FFFFFF, opacity 0.94
Tint             tinted glass only: accent, Linear Burn
Glass Effect     GLASS effect + inner shadows (rim light, top shade); own paint = white Multiply (no-op)
content          vibrant labels: Linear Burn (light) / Linear Dodge (dark)
```

Per pixel: backdrop → tone (fills) → refract + frost + disperse + highlight → inner shadows → outside: shadow + burn hairline → content.

Consequences:
- Tint = separate pass **below** the lens.
- Rim light = additive band on top **and** bottom inner edge. Not one top-left specular.
- Outer edge = darkening (burn). Not a white stroke.
- Glass once per stack. Everything above = fill + blend mode, or solid semantic colour.

## 3. Glass recipes [Kit]

Figma `GLASS`: frost (blur radius), refraction 0–1, depth (bevel width pt), dispersion 0–1, splay 0–1, light angle, light intensity. Light angle 0° everywhere.

| Recipe | frost | refr | depth | disp | splay | light | Used by |
|---|---|---|---|---|---|---|---|
| **Regular small** | 6 | 0.7 | 30 | 0.2 | 0.2 | 0.25 | buttons, toolbars, tab bars, search, accessory bars, macOS toolbar controls |
| **Regular medium / large** | 16 | 0.7 | 30 | 0.2 | 0.2 | 0.25 | menus, sheets, alerts, context menus, pickers |
| **Clear** | 6 | 0.7 | 30 | 0.2 | 0.2 | 0.40 | Control Center modules, Lock Screen buttons, collapsed notifications |
| **Dock / Home search pill** | 3 | 0.7 | 30 | 0.2 | 0.2 | 0.20 | iPhone/iPad Dock |
| **Keyboard** | 8 | 0.8 | 40 | 0.5 | 0 | 1.0 | keyboard plate (keys are fills) |
| **Glass text** | 0 | 0.5 | 1 | 0 | 0.2 | 0.40 | Lock Screen clock digits |
| **Pressed knob lens** | 0 | 0.8 | 7 | 0 | 0 | 0 | switch/slider knob while held (macOS: refr 0.9, depth 4, splay 0.2) |
| macOS menu | 25 | 0.7 | 40 | 0.4 | 0.2 | 0.20 | menus |
| macOS popover | 26 | 0.7 | 40 | 0.2 | 0.4 | 0.15 | popovers |
| macOS notification | 25 | 0.7 | 30 | 0.2 | 0.2 | 0.25 | banners |
| macOS Dock | 16 | 0.7 | 30 | 0.2 | 0.2 | 0.10 | Dock |

Variables: `Liquid Glass/Frost - Regular` 6, `Frost - Medium` 16, `Frost - Large` 16. Only frost is a variable.

Read it as:
- **One lens** (0.7 / 30 / 0.2 / 0.2). Size changes frost and tint, never the lens.
- **Frost by role**: 3 Dock, 6 controls, 16 panels, 25 macOS floating panels.
- **Clear = Regular-small lens + brighter highlight + almost no tint.**
- **Pressed lens = pure refraction.** Frost 0, highlight 0, 9 % white fill.

Parameter meaning:

| Figma | Plugin API | Meaning |
|---|---|---|
| Frost | `radius` | backdrop blur radius; Gaussian σ ≈ radius / 2 [D] |
| Refraction | `refraction` | bevel displacement strength |
| Depth | `depth` | **bevel width in pt**: how far the curved edge reaches inward. 30 on a 48 pt capsule = whole dome |
| Dispersion | `dispersion` | chromatic split inside the bevel |
| Splay | `splay` | how far the highlight spreads round the rim |
| Light | `lightAngle`, `lightIntensity` | built-in highlight; kit adds its own rims as inner shadows |

## 4. Material stacks [Kit]

Notation: `(x, y) blur b spread s colour blend`.

### Regular small

| Layer | Light | Dark |
|---|---|---|
| Fill | `#000000@25%` + `#FFFFFF@25%` + `#444444@60%` Linear Dodge + `#F8F8F8@20%` Luminosity | `#999999@17%` Luminosity |
| Drop shadow | (0, 8) blur 15 black 2 % | black 4 % |
| Outer hairline | (0, 0) blur 0 spread 0.5 `#E8E8E8` Linear Burn | `#A6A6A6` Linear Burn |
| Side hairlines | (±1.25, 0) blur 0 spread −0.75 `#D0D0D0` Linear Burn | `#A6A6A6` |
| Top shade (inner) | (0, 40) blur 30 spread −40 `#E6E6E6` Linear Burn | same |
| Rim light (inner) | (0, −40) and (0, 40) blur 10 spread −40 `#282828` Linear Dodge | `#1A1A1A` Linear Dodge |

Shadow under a control ≈ nothing. Edge = hairline + rim.

### Regular medium / large

| Layer | Light | Dark |
|---|---|---|
| Fill | `#FFFFFF@70%` Lighten + `#BFBFBF@10%` Darken | `#1A1A1A@70%` Luminosity + `#1A1A1A@90%` Luminosity + `#1A1A1A` Lighten |
| Drop shadow | (0, 8) blur 48 black 25 % | black 45 % |
| Hairlines | `#DBDBDB` Linear Burn, geometry as small | `#A6A6A6` |
| Top shade | off | off |
| Rim light | `#282828` Linear Dodge | `#1A1A1A` |

70 % white Lighten clamps everything behind to light grey or brighter. **Panel legibility = tint, not blur.**

### Clear

Fill `#101010` Linear Dodge + `#FFFFFF@4%` Luminosity. Hairline `#CCCCCC`, sides `#D0D0D0`. Top shade + rims as Regular small. No dimming in the kit: host adds it (`design-rules.md` §1).

### Tinted (prominent)

`Fill + Shadow`: shadows + side hairlines `#E6E6E6` only. **White Backing** `#FFFFFF` opacity 0.94. **Tint** accent (`#0088FF`) Linear Burn. Then Regular-small glass effect. Label white Linear Dodge.

→ Tinted button = **near-opaque colour with a glass rim**. Backdrop contributes 6 %. Tab-bar tint: same at 50 % white.

### Dock / search pill

Fill `#999999@33%` Luminosity. Extra inner (0, ±1.25) blur 0.25 `#282828` Linear Dodge = crisp 1 pt line top + bottom over the soft 10 pt rim band.

### Keyboard

Plate: `#D4D4D4@74%` Luminosity + `#1B1B1B` Linear Dodge + `#E6E9ED` Linear Burn; inner (0, 0) blur 8 black 5 %. Keys = fills `#191919` Linear Dodge + `#FFFFFF@67%`.

### Glass text

Glyph outlines carry the effect; under them a copy filled `#404040` Linear Dodge with background blur 80; inner (0, ±1) blur 3 spread −40 `#282828` Linear Dodge. Depth 1 → only the outline lenses.

### Pressed knob lens

Fill `#FFFFFF@9%`. iOS: 60 × 40 over a 38 × 24 knob (×1.58, ×1.67). macOS: 50 × 31 over 32 × 20, plus 0.5 stroke `#CCCCCC` Linear Burn, inner (0, ±3) blur 0.5 spread −2.25 `#282828` Linear Dodge ("Specular Light"), blurred 1 pt black stroke at 50 % ("Shadow"). On device: ×1.68 wide, ×1.57 tall; track seen through it ≈ 12 % smaller (lens minifies) [M5].

## 5. On top of glass [Kit]

| Element | Built as |
|---|---|
| Label, symbol | `Labels - Vibrant/Primary` `#1A1A1A` (dark `#F5F5F5`), Linear Burn / Linear Dodge |
| Secondary, tertiary | `#727272` / `#BFBFBF` (dark `#8A8A8A` / `#404040`) |
| Selection (tab, segment, row, hovered item) | capsule `Fills - Vibrant/Tertiary` `#EDEDED` (macOS Secondary `#E6E6E6`), Linear Burn. **Not glass** |
| Sub-button in a sheet/alert | capsule `#787880@16%` Linear Burn |
| Default button in alert | `#FFFFFF@60%` + accent Linear Burn |
| Separator | `#E6E6E6` Linear Burn (dark `#1A1A1A`) |
| Active toggle in a panel | opaque system colour circle, white glyph |
| Lock Screen widget text | white 60–100 % Linear Dodge over background blur 40 |

Vibrant colours = greys meant to be blended. Burn of `#1A1A1A` → near-black that keeps the hue behind it. Web: `mix-blend-mode: plus-darker` / `plus-lighter` (fallback `multiply` / `screen`).

## 6. States [Kit]

| State | Change |
|---|---|
| **Inactive** (window not key / `Active = False`) | **glass removed**: flat `#747480@8%` (light) / `#767680@24%` (dark) + 0.5 hairline |
| Disabled | glass stays, content → tertiary label |
| Selected | burn capsule inside glass |
| Pressed knob | §4 lens |
| Focused field | accent caret 2 pt, r 2 |

## 7. Corners

Continuous curvature (Figma smoothing 0.6) on every glass rounded rectangle and every fill on glass [Kit]. Corner with radius `r`, smoothing `s` reaches `(1 + s)·r` along each edge; shrinks when it does not fit half the short side → circles and capsules stay round.

Per-corner radii exist: sheet 38 top / 58 bottom, keyboard 27 / 62, iPhone Duo keyboard 24 / 24 / 54 / 5 (bottom follows display corner). Shape needs four radii.

Typical: capsule (controls), 34 (menus, alerts, edit menu), 30 (quick actions, CC 2×2), 23–26 (banners, action groups), 20 (menu control cell, macOS popover/notification), 12 (macOS menu), 16 (macOS window).

Never mix circular and continuous corners in one cluster: rim highlight shows the kink.

## 8. Compositor (Core Animation, macOS 26) [RE]

`CABackdropLayer` + private `glassBackground` CAFilter fed by an SDF layer (`CASDFLayer`, union, smoothness 8) + two `CASDFGlassHighlightEffect`. Captured on a capsule control:

| Group | Parameter | Value | Reading |
|---|---|---|---|
| Backdrop | `scale` | 0.25 | sampled at quarter resolution |
| | `tracksLuma`, `allowsFilteredLuma` | true | adaptivity reads backdrop luminance |
| Blur | `inputBlurRadius` | 1.5 (≈ 6 pt at 0.25 scale) | frost |
| Refraction | `inputInnerRefractionAmount` / `Height` | −40 / 12.5 | inward pull inside a 12.5 pt band |
| | `inputOuterRefractionAmount` / `Height` | 10 / 6.25 | slight lensing outside the edge |
| Face | `inputFaceColorMatrixBlack` / `White` / `Saturation` | 0.1 / 0.3 / 1 | dynamic-range compression |
| | `inputFaceColorMatrixFillColor` | white α 0.02 | hook for `.tint()` |
| Shadow | offset / blur / opacity | (0, 8) / 40 / 0.4 | luma-aware |
| Highlight | ×2 | angles +45° and −135°, amount 0.5, curvature 1.7, spread 1.59 | two opposite rim lights |
| Highlight tone | `vibrantColorMatrix` | saturation ≈ 2.8, gain 1.45 | rim = brightened, saturated backdrop copy. Not flat white |

## 9. Adaptive behaviour [Apple]

1. Lensing defines shape; blur secondary.
2. Bigger = thicker: deeper shadow, more frost, more opaque face.
3. Shadow stronger over busy content, lighter over flat light.
4. Small elements flip light/dark; large ones follow the scheme (`design-rules.md` §5).
5. Interaction: flexes, lights from the finger.
6. Materialize by lensing, never alpha.
7. Shapes in one container blend as they approach, morph between states.
8. Inactive window: glass flattens (§6).

## 10. Looks right?

- [ ] Backdrop bends within ≈ 10–30 pt of edge; centre flat and colour-clean.
- [ ] Bright rim top **and** bottom. Not a uniform border.
- [ ] Hair-thin darker line just outside the edge.
- [ ] Control interior = backdrop with tone, not a grey overlay. Panel interior = near-opaque light (or dark) face.
- [ ] Faint colour fringe only inside bevel. None on a pressed lens.
- [ ] Shadow: barely there under controls, deep and soft under panels.
- [ ] Tinted = almost solid colour, glass rim, label by contrast.
- [ ] Neighbours in a group fuse before touching. Press does not fuse them.
- [ ] Appears by lensing; no opacity cross-fade.
- [ ] Idle = 0 frames.
