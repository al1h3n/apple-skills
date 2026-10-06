# Colour and materials

## 1. Semantic system colours (iOS light / dark; iOS 26 kit values where Apple changed them)

| Role | Light | Dark | Use |
|---|---|---|---|
| Blue (action) | #007AFF (kit 26: #0088FF) | #0A84FF | primary action, links, "you are here" |
| Green | #34C759 | #30D158 | on/open/clean, toggles on |
| Red | #FF3B30 (kit 26: #FF383C) | #FF453A | destructive, errors |
| Orange | #FF9500 | #FF9F0A | warnings |
| Yellow | #FFCC00 | #FFD60A | ratings, people |
| Purple | #AF52DE | #BF5AF2 | one category |
| Label | #000000 | #FFFFFF | primary text |
| Secondary label | #3C3C43 @ 60 % | #EBEBF5 @ 60 % | secondary text |
| Separator | #3C3C43 @ 29 % | #545458 @ 60 % | hairlines |
| Grouped background | #F2F2F7 | #1C1C1E | solid panels (glass off) |

Every custom colour needs four variants: light, dark, light + increased contrast, dark +
increased contrast.

## 2. Discipline

- One accent per screen. Category colours are data, not decoration.
- Colour on the **glyph** to show meaning; colour on the **background** only for the primary
  action; never a solid fill on glass (use the tint path).
- Text contrast ≥ 4.5 : 1 (≥ 3 : 1 for ≥ 18 pt or bold ≥ 14 pt and for icons).
- Don't communicate with colour alone: add a label, icon or position.

## 3. Light / dark / glass on / off as a system

Build four looks from one structure: Theme (Dark / Light) × Material (Liquid Glass / Solid).

| Token | Dark glass | Light glass | Dark solid | Light solid |
|---|---|---|---|---|
| Panel face | #1C1C1E @ 42 % + glass | white 70 % lighten + #BFBFBF 10 % darken + glass | #1C1C1E @ 97 % + shadow | #F2F2F7 @ 98 % + shadow |
| On-panel fill | white 12 % | black 6 % | white 12 % | black 6 % |
| Selected | white 24 % | white 92 % (raised) | white 24 % | white 92 % |
| Label | white | black 88 % | white | black 88 % |

Glass details (refraction, rim, merging, Clear vs Regular): use **liquid-glass-skill**.
Glass belongs to the functional layer only; one glass per stack; Clear only over imagery.

## 4. Settings people expect

- Theme: System / Light / Dark (System default).
- Liquid Glass on/off plus a transparency slider (ultra clear ↔ tinted), honouring Reduce
  Transparency and Increase Contrast automatically.
