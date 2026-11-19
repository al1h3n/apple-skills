# Design rules: variants, colour, layering, legibility, accessibility

Hold on every platform (native, web, any engine). Read before choosing colours or stacking glass.

## 1. Variants

| Variant | Native API | Use | Behaviour |
|---|---|---|---|
| **Regular** | `Glass.regular` | Default. Bars, buttons, menus, sheets, anything with text | Adaptive tone. Small: flips light/dark with backdrop. Large: near-opaque face, follows scheme |
| **Clear** | `Glass.clear` | Controls over **media** (photo, video, map, wallpaper) | Never flips. Almost no tint. White content |
| **Tinted** | `.regular.tint(c)`, `.glassProminent` | The **one** primary action | Near-opaque colour (94 % white backing + accent burn [Kit]), glass rim |
| Identity | `Glass.identity` | Transition endpoint, conditional off | Draws nothing |

Clear only when all three hold [Apple]:
1. over media-rich content;
2. a dimming layer will not hurt that content;
3. content on the glass is bold and bright.

Clear over bright media → dark layer **35 %** behind it [Apple]. Skip over dark media or when the player dims. Small control → dim locally, round the control only.

Never mix Regular and Clear in one group [Apple].

### User setting

Settings → Appearance: **transparency slider, "ultra clear" → "fully tinted"** [Apple]. Native apps: free. Custom renderer: expose one 0…1 knob (0 clear, 0.5 default, 1 tinted) mapped to tone opacity + frost. Reduce Transparency → push toward 1. Clear-mode app icons ignore the slider (pinned) [Apple].

## 2. Colour

> "By default, Liquid Glass has no inherent color, and instead takes on colors from the content directly behind it." [Apple]

| Goal | Colour goes on | Example |
|---|---|---|
| Primary action | glass **background** (tinted) | Done, Buy, Send |
| Toggle state inside a glass panel | opaque system-colour **circle** behind white glyph. Not glass | Wi-Fi `#0088FF` |
| Meaning of a value | **glyph / value text**, glass neutral | brightness sun, a "good" value in blue with a check [M7] |
| Destructive row | text + symbol in system red, no background | Remove App `#FF383C` |
| Selection | burn capsule (neutral). Accent on the label only **after** it settles (+0.2 s) [M8] | camera mode strip |
| Brand | content layer | hero image |
| Bar over colourful content | nothing: monochrome glyphs | Photos |

Hard rules:
- **One tinted control per screen**, two at most. "When every element is tinted, nothing stands out" [Apple].
- Tinted control appears **when it becomes usable**, not before [M8].
- **Label on tint by contrast**: white on blue/red/green, near-black on yellow [M8]. Never fixed white.
- Emphasis → colour the background, not the symbol [Apple].
- System colours (light, dark, increased-contrast variants ship). Custom colour → supply all four.
- No flat opaque rectangle pretending to be a glass button: keep rim, hairline, shadow.
- Glyphs on Regular glass = vibrant label colours, flip with the glass. Fixed white only on Clear.

Accents [Kit]: blue `#0088FF` (dark `#0091FF`; increased contrast `#1E6EF4` / `#5CB8FF`), red `#FF383C` (`#FF4245`), green `#34C759`.

## 3. Layering

> "Always avoid glass on glass." [Apple]

| Situation | Do |
|---|---|
| Button or icon inside a glass panel | fill + vibrancy (`anatomy.md` §5), or solid semantic colour. No second glass |
| Several glass controls side by side | one container: one pass, they merge and morph |
| Selection inside a glass bar | burn capsule. Lifts into a lens only **while held/dragged**, drops back to fill |
| Menu, popover, sheet from a glass button | the button **becomes** it (one shape). Button absent while open [M7] |
| Unavoidable stack (sheet over toolbar) | upper = Regular (panel, near-opaque). Never Clear over glass |
| Glass over a dim layer | fine: dim = content |
| Full-height sheet | opaque. Only partial-height sheets are glass [Kit] |

## 4. Where glass belongs

- **Functional layer only**: navigation, toolbars, tab bars, floating buttons, partial sheets, menus, popovers. Never list rows, cards, body text, backgrounds [Apple].
- Exception: content-layer knob (slider, switch) turns into a lens **while pressed** [Apple][Kit].
- Sparingly: custom glass on the most important functional elements only [Apple].
- **Not glass in Apple's own kit**: segmented controls, steppers, text fields, lists, table rows, content-area buttons, keyboard keys, page control (Ultrathin material), iPad popovers (Thick material), macOS tooltips (Ultra Thick), macOS window sidebars (vibrancy), Lock Screen widgets (blur + dodge), full-height sheets, macOS windows [Kit].
- **Inactive window → no glass**: controls fall back to a flat fill [Kit].
- Bar changing mode keeps its title slot fixed; only buttons change [M3].
- Concentric corners: inner radius = outer − padding. Capsules for small controls, continuous corners for panels and for fills on them (`anatomy.md` §7).
- Steady state: no content/glass intersections left by accident; move the content [Apple].

## 5. Legibility

Decide **per glass element, on CPU, from backdrop luminance under it**. Never per pixel. Never guess in the host.

| Kind | Behaviour |
|---|---|
| Small Regular with glyphs (bar buttons, tab bars, capsules) | **Adaptive, ignores theme.** Whole element light with dark glyphs, or dark with light glyphs. Members of one container decide together [M8] |
| Large Regular with text (menus, sheets, banners, sidebars, alerts) | **Follows colour scheme.** Face opacity carries legibility [M4][Kit] |
| Clear | Never flips. White content. Dim backdrop 35 % when bright |
| Tinted | Tint decides; label by contrast |

Timing [M8]:

| Direction | Dwell | Glyph switch | Face |
|---|---|---|---|
| → light (white glyphs would vanish) | 0 | ≤ 33 ms | ≈ 0.25 s |
| → dark | 0.1 s | at face midpoint (Apple switches at start → 0.15 s contrast dip; do better) | ≈ 0.4 s |

- Light face only over a **clearly light** backdrop. Mid-tones → dark face, white glyphs.
- Hysteresis between two thresholds: boundary scrolling under a bar must not flicker.
- Before first sample: scheme. Snap (no fade) while the glass is still materializing.

Overrides: per glass `Adaptive | Scheme | Light | Dark`; one global lock to theme for products wanting a fixed look. Host reads the resolved tone and colours labels from it.

Also:
- Scroll edge effect under floating bars: **soft** = background blur 10 behind a gradient mask; **hard** = `#FFFFFF@85%` + blur 6, for pinned accessories under a toolbar [Kit][Apple].
- Text on glass: semibold or heavier at small sizes; vibrant label colours, never a fixed grey.
- Much text → Regular panel.

## 6. Accessibility

| Setting | System [Apple] | Custom renderer |
|---|---|---|
| Reduce Transparency | frostier, hides more | frost ×3, tone opacity high, refraction down |
| Increase Contrast | mostly black/white, contrasting border | flatten tint, 1 pt border in label colour, increased-contrast accents |
| Reduce Motion | lower intensity, no elastic | bounce 0, no press scale, no travel, cross-fades stay (`motion.md` §6) |
| Transparency slider | ultra clear → fully tinted | §1 knob |
| Web | none | `prefers-reduced-transparency`, `prefers-contrast`, `prefers-reduced-motion`, `prefers-color-scheme` |

Also: 44 pt targets on glass controls (kit toolbar buttons 44–48). Focus ring visible on glass (keyboard, pointer). Contrast checked against **both** tones the glass can take.

## 7. Anti-patterns

| Don't | Because |
|---|---|
| Glass on every card / row | competes with content |
| Tint every button | nothing stands out |
| Glass in glass | double refraction; use fills |
| Clear over plain UI | unreadable; Clear = media only |
| Regular + Clear in one bar | two materials, one group |
| Big blur + white overlay | 2020 glassmorphism |
| White 1 px border | Apple darkens the outside, lights the inside |
| One top-left specular blob | rim is top + bottom |
| Fixed white glyphs on Regular | vanish when glass goes light |
| Per-pixel light/dark text | flicker, unreadable words |
| Alpha-fade glass in/out | materialize through lensing |
| Second glass on top of the button for its menu | one shape grows |
| Neighbours bulging/fusing on press | removed in 27; noise |
| Collision physics between blobs | merge is geometric (SDF) |
| Rainbow rim on controls | dispersion stays faint, zero on pressed lens |
| Ease-in on something a finger sent | launch it (`motion.md`) |
