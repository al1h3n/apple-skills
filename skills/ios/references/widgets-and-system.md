# Widgets and system surfaces

Measured behaviour of system moments (banner, assistant island, Control Center pages, backdrop depth): liquid-glass `effects.md`. Material of each surface: liquid-glass `components.md` §5.

## 1. Home Screen widgets [Kit]

| Size | iPhone (pt) | iPad (pt) |
|---|---|---|
| Small | 164.67 × 164.67 | 136 × 136 |
| Medium | 349.67 × 164.33 | 300 × 136 |
| Large | 349.67 × 365 | 300 × 300 |
| Extra large | 349.67 × 565.67 | 628 × 300 |

Corner 28 continuous. Content margin 16 (system-provided; use `contentMarginsDisabled` only for full-bleed art).

Design:
- **One idea per widget.** Small = one fact + one glance. Medium = fact + short list. Large = detail.
- Glanceable in 2 s. No scrolling. Tap opens the matching place in the app (small: whole widget; larger: several targets ≥ 44).
- Interactive controls (buttons, toggles) for one-step actions only.
- Text ≥ 11 pt; Dynamic Type honoured; numbers tabular.
- **Rendering modes**: full colour; **accented / tinted** (system recolours: mark what keeps colour, the rest becomes one tint); **clear** (glass tile: content must work as near-white on any wallpaper). Supply luminance-first artwork: strong shapes, no reliance on hue.
- Glass widgets: tile is the glass; content sits above it and does not refract. Rim takes the wallpaper's colour.

Animated content (weather, timers, live values):
- Animate the **content**, not the tile. Tile stays still.
- Looping ambient motion slow (≥ 3 s per cycle), low contrast, pausable; static under Reduce Motion and Low Power.
- A value that changed while unseen: show the new value, no replayed animation. Changed while watched: one short transition (numbers roll once).
- Timeline entries, not timers, drive updates; budget is limited.

## 2. Lock Screen [Kit]

| Element | Spec |
|---|---|
| Clock | glass text (glyph outlines are the lens); resizable |
| Inline widget (above clock) | 327 × 26, one line, symbol + text |
| Circular | 72 × 72 (58 visible) |
| Rectangular | 160–169 × 72 |
| Bottom buttons | 58 × 58 Clear glass, 194 apart |

Lock Screen widgets are **not glass**: blurred backdrop + additive white (60–100 %). One tint, no colour. Gauges: open or closed ring, bar. Legible over any wallpaper → bold weights, no thin strokes.

Notifications rise from the **bottom centre** and stack upward. Live Activities sit at the bottom above the buttons.

## 3. Notifications [Kit]

- Collapsed: 386 × 66, r 23–24, Clear glass; icon 38; title Semibold 15, one or two lines of body; time secondary top-right.
- Expanded: opaque content card r 26 + a glass action menu.
- Banner on the Home Screen comes from under the clock and returns there.
- Write: who / what in the title, the useful fact in the body, no app name repeated. Actionable without opening where possible (2–4 actions).
- Group by thread. Summaries for low priority. Time Sensitive only when it is.

## 4. Dynamic Island and Live Activities

- Island at rest 95 × 36.67 (kit; ≈ 126 × 37 on the larger hardware cut-out).
- Presentations: compact (leading + trailing, either side of the camera), minimal (one glyph when several activities), expanded (long-press: up to ≈ 160 tall).
- Content hugs the island: concentric with its shape, no content crossing the camera.
- One fact per side. Progress as a ring or bar. Colour = the activity's key colour, sparingly.
- Grows and shrinks as one shape from the island; never slides in from elsewhere.
- Lock Screen version of the same activity: full-width card.

## 5. Control Center [Kit]

- Modules: Clear glass. 70 × 70 circle (1×1), 155 × 70 capsule (2×1), 155 × 155 r 30 (2×2); iPad 66 / 142 r 34. Padding 14.
- Custom controls (apps): button or toggle; symbol + short title + optional value. State colour only in the small icon circle; title white.
- Pages stacked vertically (controls, Now Playing, Connectivity). Edit mode: resize by a corner grip, drag to reorder.
- Same control can live on the Lock Screen and the Action button: design the symbol to stand alone.

## 6. Home Screen, icons, Dock

- Icons 64 × 64 on the 402-wide grid; label Medium 12 white with a soft shadow.
- Dock 368 × 103 r 38, thinnest glass (frost 3); search pill 77 × 30 above it.
- Icon appearances: default, dark, clear, tinted. Design per liquid-glass `icons.md`.
- Quick actions (long-press an icon): glass menu 250 wide r 30, ≤ 4 app actions + system ones.

## 7. System sheets you do not redesign

Share sheet (activity view), photo picker, document picker, Sign in with Apple, payment sheet, Face ID prompt (402 × 190 area at the top), permission alerts. Call the system one. For permission alerts: ask at the moment of need, with a purpose string that states the benefit.

## 8. Keyboard

- Plate is glass; keys are fills. Heights 307–392 by type; letter layout 332 including the suggestion row.
- Keyboard type per field (email, URL, number pad, phone). Return key label = the action (Search, Go, Send, Done).
- Layout guide: content above the keyboard stays reachable; bottom bars ride on top of it or hide.
- Accessory bar 48 for field navigation / formatting.
