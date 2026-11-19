# macOS

What differs from iPhone / iPad. Sizes and material stacks: liquid-glass `components.md` §6–8 [Kit, "macOS 27"].

## 1. Density

- Body SF Pro 13/16; control labels **Medium** 13. Sidebar rows 11.
- Control heights: **Medium 24**, **XL 36** (toolbar). Push button 67 × 24 minimum.
- Pointer targets ≥ 24; rows 20 (table), 24 (sidebar small, menu item).
- Window content margin 20. Form: labels right-aligned, secondary colour, controls left-aligned in one column.

## 2. Where glass is

| Glass | Not glass |
|---|---|
| Toolbar / titlebar controls: buttons, button groups, segmented control, pop-up, pull-down, search (Regular small capsules) | Window body (opaque, r 16) |
| Menus (frost 25, r 12) | Content-area controls: push buttons (`#000000@8%` fill), text fields, steppers, checkboxes, radios, content segmented control (solid accent selection) |
| Popovers (frost 26, r 20) | Sidebar inside a window (vibrancy: blur 80 + luminosity + saturation layers) |
| Alerts (Regular large, r 34) | Tooltips (Ultra Thick material) |
| Notifications (frost 25, r 20) | Tables, lists, group boxes |
| Dock (frost 16, r 15) | Menu bar (no background; text over wallpaper) |

Floating panels are frostier than on iOS (25 vs 16): they sit over dense windows, not over a wallpaper.

## 3. Window

- r 16 continuous. Shadow (0, 18) blur 54 black 30 % + 1 pt ring at 80 %.
- Title bar 31 tall; unified toolbar 52. Title Bold 13, subtitle Medium 11 secondary.
- Window controls: 14 pt beads, gap 9, 19 from the leading edge in a toolbar; utility panels 10 pt.
- Types: default, unified toolbar, unified compact; each with or without title, with or without sidebar.
- **Inactive window**: glass controls become flat fills; window controls grey; labels dim. Design it.
- Scroll edge under the toolbar: hard style (`#FFFFFF@85%` + blur 6), 32 tall.

## 4. Sidebar

- 210–240 wide. Rows 24 / 32 / 36 (Small / Medium / Large, user setting). Symbol in accent, label Medium 11, detail tertiary.
- Selection: neutral capsule r 8 (`#F2F2F2` Linear Burn). Not accent-filled.
- Section header Bold 11 secondary, collapsible chevron.
- Sidebar toggle = glass button in the toolbar.

## 5. Menus

- 244 wide typical, r 12, padding 12 h / 5 v. Item 24 tall, 13 pt; shortcut right-aligned, tertiary; submenu chevron.
- Header rows Bold 13 secondary. Separators hairline burn, inset.
- Every command in the menu bar; toolbar and context menus repeat a subset. Keyboard shortcut for every frequent command.
- Menu from a toolbar button grows out of that button (one shape).

## 6. Controls that differ from iOS

| Control | macOS |
|---|---|
| Switch | 54 × 24; **on = accent**, not green; knob 32 × 20 |
| Checkbox / radio | exist (16 × 16); use for settings lists instead of switches when several apply at once |
| Segmented control | toolbar: glass capsule with burn selection; content: r 6, solid accent selection |
| Pop-up vs pull-down | pop-up shows the current choice (double chevron); pull-down is a menu of actions (single chevron) |
| Stepper | 20 × 24, beside its field |
| Slider | knob 20 × 20; dial 24 × 24 |
| Alert | 260 wide, icon 64 on top, stacked or side-by-side buttons 228 × 28 capsules; default = solid accent |
| Tooltip | 11/13, appears after a hover delay; never the only place for essential info |

## 7. Pointer and keyboard

- Hover is a state: design it (subtle fill or scale ×1.03 on glass). Pressed distinct from hover.
- Focus ring on every control; full keyboard navigation; Escape closes the top surface; Return = default button.
- Right-click context menu wherever a selection exists.
- Drag and drop: lifted item ×1.08, drop target highlights.
- Resizable everything: test minimum window size and very wide.

## 8. Desktop apps that are not native (Tauri, Electron, web)

- Same density and structure. System font stack.
- Custom titlebar: keep the platform's window-control position and behaviour; draggable region; double-click zoom.
- Glass over app content only (the app owns that canvas). Never fake refraction of the desktop behind a web view.
- Honour Reduce Transparency / Motion / Contrast through media queries.
- Windows and Linux builds: keep the hierarchy and craft, drop Mac-only chrome (traffic lights, menu-bar assumptions).
