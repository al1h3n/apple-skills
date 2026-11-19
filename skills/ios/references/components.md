# Components: which one, and how it behaves

Sizes, radii, glass or fill for every control and bar: liquid-glass `components.md` §2–5 (from Apple's kit). This file: choosing, and what is not there.

## 1. Choose the control

| Need | Use | Not |
|---|---|---|
| The one next step | glass prominent (tinted) button; in content: solid accent capsule | two tinted buttons |
| Secondary action on a bar | glass button, symbol or short text | text + symbol crowded |
| Action inside content | bordered / borderless content button (fill, no glass) | glass in content |
| On / off, takes effect at once | switch | checkbox (macOS thing) |
| 2–4 exclusive views of the same content | segmented control | tabs |
| One of many options | menu from a pop-up button; ≥ 8 with search → pushed list | wheel picker for short lists |
| A value on a range | slider (ticks when stops matter) | stepper for wide ranges |
| Small integer | stepper | slider |
| Date / time | compact chips opening the calendar picker | custom calendar |
| Few commands on an object | context menu (long-press) + visible "…" menu | hidden gesture only |
| Confirm a destructive act | action sheet (from the control that asked) or alert | alert for non-critical info |
| Critical interruption | alert: title, one line, ≤ 3 buttons | long text, forms |
| Supplementary task | sheet (medium detent if context matters) | full-screen cover for small tasks |
| Transient info | banner / inline status | modal |
| Progress of known length | progress bar (4 tall) | spinner |
| Unknown length, small area | spinner; content area → skeleton | full-screen spinner |
| Nothing to show | empty state: symbol or title, one line, one glass button (330 × 52, margins 36) | blank screen |

## 2. Lists [Kit]

| Part | Spec |
|---|---|
| Row | 52 tall (68 with two lines); padding 16; title Body 17/22; detail secondary; chevron tertiary |
| Leading image | 52 × 52; icon badge 22 circle accent |
| Group | inset: margin 16, r 26, one fill; hairlines inset to the text |
| Section header | padding 16 h; nested 42, prominent 45, extra prominent 48 tall |
| Section title (in menus) | 29 tall, secondary |
| Footer | padding 16 h, 8 top, 6 bottom; Footnote secondary |
| Swipe actions | capsules 60 × 44, solid colours, symbol only or symbol + label; row content becomes a capsule while swiped |
| Index bar | 16 wide (iPad 30); letters Medium 12 accent |
| Edit mode | leading selection circle 22; reorder grip trailing |

Rows are content: **never glass**. Background grouped `#F2F2F7` (dark `#000000`), rows `#FFFFFF` (dark `#1C1C1E`); inside a sheet (elevated) dark `#1C1C1E` / `#2C2C2E`.

## 3. Text input

- Field row 52, padding 16, value Medium 17; placeholder tertiary. Inside an alert: stacked in one r 26 group.
- Caret accent, 2 wide. Selection handles + edit menu (glass capsule 44 tall: Cut · Copy · Paste · ›).
- Keyboard: glass plate, 307–392 tall by type; accessory bar 48 above it for find / URL / form navigation (glass field + Done).
- Search: bottom toolbar on iPhone ("Search" type), top trailing on iPad (180 × 44). Sidebar search 44 capsule fill.
- Never cover the focused field with the keyboard; scroll it into view.

## 4. States every control needs

rest · pressed · disabled · focused (hardware keyboard, pointer on iPad) · selected · loading. Glass specifics: liquid-glass `anatomy.md` §6. Pressed timing: liquid-glass `motion.md` §3.

## 5. Behaviours

| Component | Behaviour |
|---|---|
| Switch, slider | knob becomes a clear lens ×1.6 while held; value colour appears inside the lens first |
| Segmented control | selection slides under the finger; fill, not glass |
| Tab bar | minimises on scroll down (to one tab + accessory), restores on scroll up; adaptive light/dark |
| Toolbar | items in one group share a capsule; mode change morphs buttons, title stays |
| Menu | grows from its button as one shape; submenu replaces in place |
| Sheet | follows the finger; snaps to detents; large detent turns opaque |
| Alert | centred, dims content 20 %; default button tinted fill; destructive red label |
| Context menu | preview lifts (r 30, shadow), background dims, menu beside it |
