# Colour and materials

Glass itself (recipes, stacks, legibility flip): liquid-glass `anatomy.md` and liquid-glass `design-rules.md`.

## 1. System colours [Kit, 27]

| Role | Light | Dark | Use |
|---|---|---|---|
| Blue | `#0088FF` | `#0091FF` | primary action, links, selection |
| Blue, increased contrast | `#1E6EF4` | `#5CB8FF` | |
| Red | `#FF383C` | `#FF4245` | destructive, errors |
| Green | `#34C759` | `#30D158` | on, open, success |
| Orange | `#FF9500` | `#FF9F0A` | warning |
| Yellow | `#FFCC00` | `#FFD60A` | ratings; needs a dark label when used as a tint |
| Label | `#000000` (macOS 85 % black) | `#FFFFFF` | primary text |
| Secondary label | `#3C3C43@60%` | `#EBEBF5@60%` | secondary text |
| Tertiary label | `#3C3C43@30%` | `#EBEBF5@30%` | placeholders, chevrons |
| Fill primary / secondary / tertiary | `#787878@20%` / `#787880@16%` / `#767680@12%` | (system dark fills) | tracks, fields, chips |
| Separator | `#3C3C43@29%` | `#545458@60%` | hairlines |
| Grouped background | `#F2F2F7` | `#1C1C1E` | solid panels |
| Overlay behind sheets | `#000000@20%` | | |

On glass use the **vibrant** set (greys blended Linear Burn in light, Linear Dodge in dark): primary `#1A1A1A` / `#F5F5F5`, secondary `#727272` / `#8A8A8A`, tertiary `#BFBFBF` / `#404040` [Kit].

Custom colour → four variants: light, dark, light + increased contrast, dark + increased contrast.

## 2. Discipline

- One accent per screen. Category colours = data, not decoration.
- Colour on the **glyph** for meaning. Colour on the **background** only for the primary action.
- Label on a tinted background by contrast: white on blue / red / green, near-black on yellow.
- Text contrast ≥ 4.5 : 1 (≥ 3 : 1 for ≥ 18 pt, bold ≥ 14 pt, icons). Check against every tone the surface can take.
- Never colour alone: add label, icon or position.
- Saturation budget: product imagery and the one action get it. Chrome stays neutral.
- Dividers: separator colour at hairline width. Never label-dark.

## 3. Material ladder

| Layer | Material |
|---|---|
| Content (lists, cards, text, fields) | opaque or standard materials. **Never Liquid Glass** |
| Controls + navigation floating above content | Liquid Glass (Regular; Clear over media) |
| Small overlays in content (page dots, tooltips, iPad popovers) | standard materials: ultra-thin … ultra-thick blur + tint |
| Selection inside glass | neutral burn capsule |
| Inactive window | glass off, flat fills |

One glass per stack. Full-height sheets opaque.

## 4. Four looks from one structure

Theme (Light / Dark) × Material (Liquid Glass / Solid). Both materials fully designed: solid is not a degraded mode.

| Token | Light glass | Dark glass | Light solid | Dark solid |
|---|---|---|---|---|
| Panel face | 70 % white Lighten + lens | 70–90 % `#1A1A1A` + lens | `#F2F2F7@98%` + shadow | `#1C1C1E@97%` + shadow |
| Control face | translucent, adaptive | translucent, adaptive | `#767680@12%` | `#767680@24%` |
| On-panel fill | `#787880@16%` burn | white 12 % | black 6 % | white 12 % |
| Selected | `#EDEDED` burn capsule | white 24 % | white 92 % raised | white 24 % |
| Label | `#1A1A1A` burn | `#F5F5F5` dodge | black 88 % | white |
| Edge | burn hairline + rim light | same | 0.5 hairline separator | same |
| Shadow, control / panel | 2 % / 25 % blur 48 | 4 % / 45 % | 8 % blur 12 / 18 % blur 40 [D] | same, darker |

Glass off (user choice, low-end device, Reduce Transparency): every other rule in this skill still applies. Only the material changes.

## 5. Settings people expect

- Theme: System / Light / Dark (System default).
- Liquid Glass: on / off + transparency slider (ultra clear ↔ fully tinted). Reduce Transparency and Increase Contrast honoured automatically.
- Reduce Motion honoured automatically; an in-app override may only reduce further.
