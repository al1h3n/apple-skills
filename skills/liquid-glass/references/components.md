# Components: per-control spec

Sizes in pt, from Apple's 27 kits [Kit] unless tagged. Recipes (Regular small/medium, Clear, …) = `anatomy.md` §3–4. Motion = `motion.md` §3. `r` = corner radius; "capsule" = radius ≥ half height.

Glass column: **RS** Regular small, **RM** Regular medium/large, **CL** Clear, **T** tinted, **none** not glass.

## 1. Rules across all components

- Control height: iOS toolbar buttons 44 (top) / 48 (bottom, tab bar); iOS text buttons 28 / 34 / 50; macOS 24 (Medium) / 36 (XL).
- Hit target ≥ 44 on touch, whatever the drawn size.
- Selection inside glass = burn capsule (`#EDEDED` iOS, `#E6E6E6` macOS, Linear Burn). Never glass.
- Group of glass buttons = one capsule, glyphs 12–20 apart inside.
- Inactive window → every RS control becomes a flat fill (`anatomy.md` §6).
- Labels on glass: vibrant primary; SF Pro Medium 17 (iOS), Medium 13 (macOS).

## 2. iOS / iPadOS controls

| Component | Size | Glass | Detail |
|---|---|---|---|
| Glass button, text | 60 × 28, 69 × 34, 85 × 50 | RS / T | capsule; padding 20 h, 16 v (large); label Medium 17/18 |
| Glass button, symbol | 50 × 50 | RS / T | symbol Semibold 19 |
| Content-area button | 28 / 34 / 50 tall | none | prominent = solid accent capsule, white label Regular 15; bordered = tinted fill; borderless = text |
| Switch | 64 × 28 | lens while pressed | on `#34C759`; knob 38 × 24 white, 2 inset; lens 60 × 40 |
| Slider | row 400 × 52 | lens while pressed | track 280 × 6 r 3 `#787878@20%`; filled part accent; ticks 4 × 4; knob 38 × 24 |
| Segmented control | 370 × 50, 370 × 32 | none | track `#767680@12%` capsule, padding 2, gap 4; selected `#FFFFFF` + shadow (0, 2) blur 20 black 6 %; labels 13.33/18 Semibold (selected) / Medium |
| Stepper | 92 × 32 | none | halves 46 × 32, separator 1 × 24 |
| Text field | row 402 × 52 | none | padding 16; value Medium 17/20; placeholder tertiary. In alert: group r 26 `#787880@16%` Linear Burn |
| Date chips | 34 tall | none | capsules `#767680@12%`, padding 11 / 6 |
| Date picker | 370 × 377, r 13 | RM | day cell 38; today `#0088FF@12%`; selected = solid label colour, inverted text; weekday Semibold 13 tertiary |
| Popup button | 65 × 44 | none | accent label Semibold 18 + chevron |
| Colour well | 28, 36 | none | angular rainbow gradient disc |
| Page control | dots 8, gap 8 | none | Ultrathin material capsule |
| Progress bar | track 4 | none | |
| Swipe actions | 60 × 44 capsules | none | solid blue / grey / green; row content capsule r 100 |
| List row | 52 (68 tall) | none | padding 16; Body 17/22; detail secondary |

## 3. iOS / iPadOS bars

| Component | Size | Glass | Detail |
|---|---|---|---|
| Tab bar, iPhone | 402 × 95 (88 minimised) | RS | bar 188 × 62 for 2 tabs; tab 92 × 53.5, tabs overlap 8; selection 96 × 54 capsule; symbol Semibold 18/28, label Semibold 10/12; bottom inset 25, top 16. Types: default, search role (separate round button), prominent tab (tinted) |
| Tab bar, iPad | 491 × 44 | RS | padding 4; tab 74–76 × 36, padding 18 / 8; label Medium 15/20, selected Bold accent |
| Tab bar, iPhone Duo | 48 wide, vertical | RS | 48 × 110 (2 tabs) … 48 × 270 (5); tab 48 × 48 |
| Toolbar top, iPhone | 402 × 54 | RS | buttons 44 × 44; side padding 16, bottom 10; groups share a capsule, glyph gap 12–20; title Semibold 17 or 15 + subtitle 12 |
| Toolbar top, iPad | 820 × 54 (98 / 103 / 131 with large title or tab bar) | RS | buttons 44 × 44; search 180 × 44; window controls: 27 × 22 glass capsule, 6 pt lights |
| Toolbar bottom, iPhone | 402 × 84 | RS | buttons 48 × 48, gap 12; side padding 28, bottom 32; variants: buttons, + page dots, search, search + item |
| Accessory bar (find, search, URL) | 402 × 48 | RS | field 330 × 48 + Done 48 × 48; gap 8, side padding 8 |
| Sidebar, iPad | 320 wide | panel behind; rows none | rows 44, padding 10 / 8, gap 8; selection burn capsule; search 44 capsule `#787880@16%` Linear Burn; toolbar buttons 44 RS |
| Scroll edge, soft | none | none | background blur 10 under a gradient mask |
| Scroll edge, hard | none | none | `#FFFFFF@85%` + blur 6 |
| Status bar | 402 × 62 | none | Dynamic Island 95 × 36.67 |
| Home indicator | 144 × 5 | none | |

## 4. iOS / iPadOS presentations

| Component | Size | Glass | Detail |
|---|---|---|---|
| Sheet, medium detent | 390 × 459, inset 6 | RM | r 38 top / 58 bottom; overlay `#000000@20%`; grabber 60 × 4 `#CCCCCC` Linear Burn, 6 from top; title Semibold 17 |
| Sheet, full height (activity view) | 402 × 984 | none | **opaque** elevated background, r 38 top; shadow (0, 15) blur 75 black 18 %; action groups r 26 `#787880@16%`; round buttons 66 `#EDEDED` Linear Burn |
| Colour picker | 386 × 599, margin 8 | RM | r 32 top / 58 bottom |
| Alert / action sheet | 300 wide, padding 14, r 34 | RM | title + message padding 8 / 24; buttons 272 × 48 capsules, gap 10: plain `#787880@16%` Linear Burn; default `#FFFFFF@60%` + accent Linear Burn; destructive label red. Side-by-side: 128 wide, gap 16 |
| Menu | 238–250 wide, r 34 | RM | item 40 tall, 17/20, padding 10 v; control group row 56 tall with r 20 cells (symbol 13 + label 12); separators `#E6E6E6` Linear Burn, inset |
| Context menu | menu 250, r 34 | RM | preview r 30, shadow (0, 0) blur 32 black 20 %; dimming overlay behind |
| Home quick actions | 250 × 350, r 30 | RM | overlay `#000000@23%` |
| Edit menu | 232 × 44 capsule | RM | text 15; separators 1 × 18 `#CCCCCC` Linear Burn; overflow chevron in a 36 burn circle |
| Popover, iPad | none | none | **Thick material**: blur 100–130, `#FFFFFF@34%` Linear Dodge + `#FFFFFF@84%` Color Dodge; shadow (0, 10) blur 100 black 30 % |
| Notification, collapsed | 386 × 66, r 23–24 | CL | padding 14 / 12; icon 38.33; title Semibold 15/17, body Regular 15/18, time secondary |
| Notification, expanded | content 370 × 370, r 26 | menu CL | content opaque; action menu 250 × 104 r 26, rows 22, gap 20 |

## 5. iOS system surfaces

| Surface | Detail |
|---|---|
| Control Center | modules CL (light 0.4): 70 × 70 circle, 155 × 70 capsule, 155 × 155 r 30; iPad 66 / 142 r 34; padding 14; state colour only in the small icon circle |
| Lock Screen | buttons 58 × 58 CL; widgets: (blur 40 + white 7–20 % Linear Dodge, text white 60–100 % Linear Dodge); clock = glass text |
| Home Screen | icons 64 × 64, label Medium 12 with shadow; grid padding 30 / 28, row gap 17.33; Dock 368 × 103 r 38 (iPad 740 × 107; Duo 72 × 264 r 24, 320 × 92 r 32); search pill 77 × 30 |
| Widget | r 28, thin inner rim shadows |
| Keyboard | plate = keyboard recipe, r 27 top / 62 bottom; keys: r 8.5, 33.5 × 43, gap 6, row gap 11; iPad keys r 13, 82 × 74 |

## 6. macOS: over glass (toolbar, titlebar)

| Component | Medium, XL | Glass | Detail |
|---|---|---|---|
| Button | 24 × 24, 36 × 36 | RS | XL: padding 4 round a 28 × 28 shape; symbol Medium 13/22 |
| Button group | 72 × 24, 100 × 36 | RS | one capsule, gap 4 |
| Segmented control | 121 × 24, 145 × 36 | RS | selected segment 32 × 28 burn capsule; separators 1 × 20 `#D9D9D9` Linear Burn |
| Pop-up button | 57 × 24, 63 × 36 | RS | padding 8 / 6.5 |
| Pull-down button | 39 × 24, 50 × 36 | RS | |
| Search | 130 × 24, 130 × 36 | RS | padding 10, gap 6; placeholder vibrant secondary |
| Window controls | 68 × 14, gap 9 | none | 14 pt beads `#FA9891` `#FFDA66` `#ADE392`, inner-shadow shading; utility panel 10 pt |

## 7. macOS: content area (none is glass)

| Component | Size | Detail |
|---|---|---|
| Push button | 67 × 24 | fill `#000000@8%`; styles bordered, tinted, destructive, prominent, borderless; clicked + `#000000@8%` |
| Switch | 54 × 24 | on = **accent** (not green); knob 32 × 20, shadow (0, 3) blur 36 black 5 %; lens 50 × 31 while clicked |
| Checkbox, radio | 16 × 16 | accent fill, white mark; radio dot 4.8; label gap 3, Medium 13/16 |
| Segmented control | 24 tall, r 6 | selected = solid accent, white label; separators 1 × 14 |
| Text field, combo box | 24 tall, r 6 | white, 1 pt ring `#000000@8%` |
| Search field | 24 tall, capsule | |
| Stepper | 20 × 24 beside 42 × 24 field | halves 20 × 12 |
| Slider knob | 20 × 20 | white; over glass white 65 % + 45 % + 35 % |
| Dial | 24 × 24 | |
| Progress bar | 6, 10 tall | track = label at 10 %; fill accent |
| Table row | 20 tall | selected `#0165E2`, r 8; text Medium 13/16 |
| Sidebar row | 24 / 32 / 36 | text Medium 11/16; symbol accent; selection `#F2F2F2` Linear Burn r 8; header Bold 11 secondary |
| Group box | r 12 | `#000000@3%` at 50 % |
| Scrollbar | thumb 6 | `#000000@50%` capsule |

## 8. macOS: floating surfaces

| Surface | Size | Material |
|---|---|---|
| Menu | 244 wide, r 12, padding 12 h / 5 v | macOS-menu glass; fill `#D9D9D9@55%` Luminosity + `#D9D9D9@40%`; shadow (0, 8) blur 48 black 25 %; item 24 tall, 13 pt; shortcut tertiary |
| Popover | r 20, arrow 10.5 | macOS-popover glass; body `#E2E2E2@35%` + blur 81.5; content padding 20, gap 8 |
| Alert | 260 wide, r 34, padding 16 h / 20 v, gap 14 | Regular large; icon 64; title Bold 13/16; message Regular 11/14; buttons 228 × 28 capsules gap 6: default solid accent, destructive `#F52F32@23%` + red label, cancel `#E6E6E6` Linear Burn |
| Notification | 344 × 77 (56 stacked), r 20 | macOS-notification glass; light fill `#BFBFBF@25%` Lighten + `#1A1A1A` Linear Dodge; dark `#333333@30%` Luminosity + `#333333@10%`; padding 10 / 12 / 14 / 12; icon 32; title Bold 13/16 |
| Dock | 1152 × 55, r 15 | macOS-Dock glass; icons 36; padding 8; gap 9; separator 1 × 36 |
| Tooltip | 18 tall, padding 6 / 3 | **Ultra Thick material**: blur 60, `#333333` Linear Dodge + `#ECECEC@88%`; text 11/13 |
| Window | r 16, smoothing 0.6 | opaque; shadow (0, 18) blur 54 black 30 % + (0, 0) blur 1 black 80 %; title bar 31; title Bold 13/15, subtitle Medium 11 |
| Utility panel | r 15 | shadow (0, 5) blur 20 black 30 % |
| Sidebar in window | 210–240 wide | **vibrancy**: blur 80, `#DAE6F2` Luminosity 92 %, `#808080` Saturation 85 % |
| Menu bar | 24 tall items | no background; white Semibold 13 over wallpaper |

## 9. macOS 27 desktop, measured [M9]

The kit rows above are Apple's Figma values; these are what macOS 27 draws on screen.

| Part | Measured |
|---|---|
| Window toolbar | **no background**: items float over the content as glass circles (one action) and capsules (a group, thin dividers), 36 pt; light capsules near-white with a soft shadow, dark smoky on dark windows |
| Scroll edge | **hard**: content under the toolbar blurred and washed toward the window colour in a band as tall as the toolbar, ending in a straight line |
| Sidebar | a floating glass panel 8 pt inside the window, corners concentric with the window's (window r ≈ 26 with a sidebar), the traffic lights in it; selection a grey rounded fill, list selection the accent with white text (r ≈ 10) |
| Traffic lights | 14 pt beads, 9 pt apart (centres 23 pt), `#FF5F57` `#FEBC2E` `#28C840`, a 0.5 pt rim at 70 % of the colour, lighter top; hover on any shows all three glyphs (×, −, two triangles) at ≈ 55 % black over the colour; inactive window: all `#DDDDDD` light / `#4D4D4D` dark, no glyphs |
| Dock | bar ≈ 66 pt for 48 pt icons, r ≈ 22, very clear, thin bright rim; magnification **by layout**: icons grow upward from the baseline and push neighbours apart (never overlap), hovered ≈ 2.4×, falloff ≈ 3 icons each side, the bar widens and keeps its height; full size in ≈ 0.1 s with ≈ 8 % overshoot; name in a small glass capsule above |
| Spotlight | "Search or Ask": ≈ 48 pt smoky capsule + a separate results panel (r ≈ 26), both heavily frosted, dark face ≈ 70 %, shortcuts at the trailing edge, microphone at the end |
| Siri (Mac) | panel ≈ 290 × 410, r ≈ 30: near black at the top fading down into heavily blurred, almost clear glass, a thin bright rim; white type throughout (large bold title, grey subtitle, a full-width action, sections under hairlines, rows with app icons); dark glass circles back / close; composer: Ask Siri capsule, mic |
| Visual Intelligence | the whole screen dims in one frame (≈ 42 % black), a dragged rectangle stays at full brightness with a thin white edge, light frosted pills (≈ 24 pt) beside it, a card grows in ≈ 0.4 s (heavy frost, r ≈ 16), spinner, answer |
| Liquid Glass setting | Clear ↔ Tinted slider, three ticks; at Clear capsules show the photo almost unblurred with a bright line inside their lower edge; at Tinted near-opaque white |

Glass that must cover other page content (Spotlight, Control Center, Siri, menus over a
window) is drawn by CSS on the web: an engine draws on a canvas under the page and cannot
see the window. Glass inside an opaque window is CSS too, for the same reason.

## 9. Behaviour by component (not in the kit)

| Component | Behaviour | Source |
|---|---|---|
| Button | press: `motion.md` §3. Opens a surface → becomes it | [M3][M7] |
| Switch, slider | knob → lens while held; colour inside lens first | [M5] |
| Segmented / mode strip | selection follows finger as a fill; re-forms round the centred item on release; accent after settle | [M8] |
| Tab bar | small Regular: adaptive tone, members flip together | [M8] |
| Toolbar | mode change: buttons morph frame to frame; title slot fixed | [M3] |
| Menu | from its button: blob → dome → menu, 0.27 s; rows inside from frame 1 | [M7] |
| Sheet | follows the finger; launched on release; content soft while moving | [M8] |
| Banner | origin = host anchor (under clock; bottom centre on Lock Screen) | [M4][M7] |
| Control Center tiles | appear together, soft → sharp 0.2 s; dismiss in place 0.18 s | [M8] |
| Tinted action | appears only when usable, rises 8 pt; label by contrast | [M8] |
| Dial | chips → arc 0.15 s; value updates in place | [M8] |
