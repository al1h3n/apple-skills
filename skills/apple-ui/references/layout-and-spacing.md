# Layout and spacing

Component sizes: liquid-glass `components.md` (iOS + macOS, from Apple's kits).

## 1. Grid and rhythm

- 4 pt base, 8 pt rhythm: 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64.
- Inside a control 8–12. Between controls in a cluster 12–16. Between groups 24–32.
- Margins: iPhone 16–20; iPad 20–24; macOS window content 20; floating panels 8 from the screen edge on phones [Kit], 24 on desktop.
- Few alignment edges. Every new left edge = a new thing to read. Same margins on every screen.
- Space connects as well as separates: related things closer than unrelated ones, always.

## 2. Targets

- Touch ≥ 44 × 44 pt. Drawn shape may be smaller; hit area may not.
- Pointer ≥ 24 pt (macOS control height), 28 preferred; 8 between adjacent targets.
- Phones: primary actions in the bottom third. Destructive ones away from the thumb's rest.

## 3. Shapes

- Continuous corners (Figma smoothing 0.6) on rounded rectangles. Capsules and circles for small controls.
- **Concentric**: inner radius = outer − padding (sheet 38 → card 26 at 12 padding).
- Per-corner radii where the display dictates (sheet 38 top / 58 bottom; keyboard 27 / 62) [Kit].

| Element | Radius [Kit] |
|---|---|
| Button, capsule, search, tab bar | height / 2 |
| Menu control cell, macOS popover, macOS notification | 20 |
| Banner, action group, text-field group | 23–26 |
| Widget | 28 |
| Quick actions, Control Center 2×2 | 30 |
| Menu, alert, action sheet | 34 |
| Sheet top | 38 |
| macOS menu | 12 |
| macOS window | 16 |

## 4. Structures

| Pattern | Use | Notes |
|---|---|---|
| Inset grouped list | settings, details | one fill per group, rows ≥ 44, hairlines inset to text |
| Segmented control | 2–4 exclusive modes | selected = raised fill; never glass |
| Sheet with detents | results, details | medium = glass, inset 6; large = opaque. Grabber 60 × 4 |
| Floating capsule cluster | map / canvas controls | one glass container |
| Sidebar / docked panel | desktop navigation, inspector | content stays visible beside it |
| Popover | small choices | anchored to its button, grows out of it |
| Bottom toolbar (phone) | 3–5 actions | symbols or text, not both crowded |
| Sticky header card | long detail pages | title + key fact stay while content scrolls |
| Sticky bottom action | purchase, confirm | primary action visible for the whole journey; quantity beside it |

## 5. Scroll

- No visible scrollbars in panels: overlay indicator (thumb 6 wide capsule, 50 % black on macOS) while scrolling.
- Scroll edge under floating bars: soft (blur 10 under a gradient) by default; hard (`#FFFFFF@85%` + blur 6) under pinned headers [Kit].
- Primary action visible without scrolling.

## 6. Responsive

- Same hierarchy, different arrangement: desktop = panels beside content; phone = sheets from the bottom; tablet = sidebar + content.
- Foldables exist (Apple's 27 kit ships "iPhone Duo"): bars may run along the leading edge (vertical tab bar 48 wide); size classes, never device idiom.
- Never scale a desktop layout down. Rebuild at phone size.
