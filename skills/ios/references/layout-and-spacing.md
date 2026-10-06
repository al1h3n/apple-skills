# Layout and spacing

## 1. Grid and rhythm

- 4 pt base, 8 pt rhythm: 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64.
- Inside a control 8–12; between controls in a cluster 12–16; between groups 24–32.
- Screen margins: iPhone 16 (compact) / 20; iPad 20–24; desktop floating panels 24 from the
  window edge.
- Align to a few edges. Every new left edge is a new thing to read.

## 2. Targets

- Touch: ≥ 44 × 44 pt (visual can be smaller; hit area cannot).
- Pointer: ≥ 28 pt; keep 8 pt between adjacent targets.
- Thumb zone on phones: primary actions in the bottom third (search bar at the bottom,
  as in iOS 26 Maps); destructive actions away from it.

## 3. Shapes

- Continuous corners (squircle, Figma smoothing 60 %) on rounded rectangles; circles and
  capsules for small controls; hardware-matched shapes (Dynamic Island) stay circular.
- **Concentric radii:** inner = outer − padding (sheet 38 → card 26 at 12 pt padding).
- Typical radii: buttons/capsules = height / 2; list groups 14–16; cards 20–28; panels 28–34;
  sheets 38–44 (iPhone corner ≈ 55).

## 4. Common Apple structures

| Pattern | Use | Notes |
|---|---|---|
| Inset grouped list | settings, details | one fill per group, rows 44 pt min, hairlines inset to text |
| Segmented control | 2–4 exclusive modes | selected = raised fill; lens while dragging (glass) |
| Sheet with detents | phone results, place details | medium / large; grabber 36 × 5; inset 8 with glass |
| Floating capsule cluster | map controls | one container, 16 apart at rest |
| Sidebar / docked panel | desktop menus, AI window | content stays visible next to it |
| Popover | small choices (sort, time) | anchored to its button, grows out of it |
| Toolbar bottom (phone) | 3–5 actions | text or symbols, never both crowded |

## 5. Scroll

- No visible scrollbars in panels: an overlay indicator (3 pt capsule, 45 % white/black)
  appears only while scrolling.
- Scroll edge effect: content fades under floating bars; pinned headers use a hard edge.
- Keep the primary action visible without scrolling (sticky footer or above the fold).

## 6. Responsive

- Same hierarchy, different arrangement: desktop = docked panels beside content; phone =
  sheets from the bottom; tablet = sidebar + content.
- Never just scale a desktop screenshot down for mobile marketing: rebuild at phone size,
  or show real phone screens at 1 : 1 so system elements (status bar, home indicator) stay true.
