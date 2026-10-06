# Design Rules: variants, colour, layering, legibility, accessibility

Rules that apply on every platform (SwiftUI, Web, Liquid Rust). Sources: HIG *Materials*
and *Color* (updated Sep/Dec 2025), WWDC25 "Meet Liquid Glass", Apple's Figma kit.

---

## 1. Variants and presets

Apple exposes two material variants plus a no-op, and a tint modifier:

| Preset | API | Use | Notes |
|---|---|---|---|
| **Regular** | `Glass.regular` / `UIGlassEffect.Style.regular` | default for everything: bars, buttons, menus, sidebars, popovers, anything with text | Adaptive: luminance compression, light/dark flip on small elements, shadow follows content. Works over any content |
| **Clear** | `Glass.clear` / `.clear` | controls floating over **media** (photos, video, maps, lock screen) | Not adaptive, permanently more transparent. Needs a dimming layer when the media is bright |
| **Tinted** | `Glass.regular.tint(color)` / `glassProminent` | the **one** primary action on screen (Done, Buy, Send) | Colour is tone-mapped to the backdrop ("stained glass"), never a flat fill |
| Identity | `Glass.identity` | animation endpoint, conditional disable | renders nothing |

Never mix Regular and Clear in one group of controls [Apple].

Use **Clear** only when all three hold [Apple]:
1. it sits over media-rich content;
2. a dimming layer will not hurt that content;
3. the content on the glass is bold and bright (white glyphs, bold labels).

Dimming for Clear: dark layer at **35 %** behind the component when the content is bright;
skip it when the content is already dark or the player supplies its own dimming [HIG].
Small controls can use *localized* dimming (just around the control) [Apple].

### User-level appearance (system setting, not an API variant)

- iOS/iPadOS/macOS 26.1: Settings → Display & Brightness → Liquid Glass → **Clear | Tinted**.
  Tinted "increases opacity and adds more contrast".
- iOS/iPadOS/macOS 27: replaced by a **slider from "ultra clear" to "fully tinted"**.
- Native apps get this for free. Custom renderers (web, Liquid Rust) should expose the same
  knob (0 = ultra clear, 0.5 = default, 1 = fully tinted) and honour
  `prefers-reduced-transparency` / Reduce Transparency by pushing it toward 1.

## 2. Colour: where to put it

> "By default, Liquid Glass has no inherent color, and instead takes on colors from the
> content directly behind it." — HIG

Decision table (answers "tint the shape or the icon?"):

| Goal | Put colour on | Example |
|---|---|---|
| Mark the **primary action** | the glass **background** (`.tint()` / prominent style), white label | Done, Checkout, "View Bag" in red |
| Show **state** of a toggle inside a glass panel | an **opaque solid circle** in the system colour behind the glyph, not glass | Control Center Wi-Fi `#0088FF`, Cellular `#34C759` |
| Convey **meaning of a value** (brightness, volume, battery) | the **glyph/fill**, glass stays neutral | sun `#FFCC00`, speaker `#00C0E8` |
| Destructive menu row | text + symbol in system red, no background | "Remove App" `#FF383C` |
| Brand presence | the **content layer**, not the glass | hero images, cards |
| Toolbar/tab bar over colourful content | nothing: keep glyphs **monochrome** (they flip black/white automatically) | Photos, Maps |
| Toolbar over monochrome content | app accent colour on glyphs is fine | Notes, Settings |

Hard rules:
- **Tint at most one or two elements per screen.** "When every element is tinted, nothing
  stands out" [Apple]. Refrain from colouring the background of multiple controls [HIG].
- To emphasise a primary action, colour the **background**, not the symbol/text [HIG].
- Don't use similar hues for labels and for a colourful backdrop.
- Use system colours (they ship light, dark and increased-contrast variants). A custom
  colour needs all four variants.
- A solid opaque fill on a glass button "breaks the visual character of Liquid Glass" —
  always go through the tint path, which maps the colour to backdrop brightness [Apple].
- Glyph colour on Regular glass should be *adaptive* (label/secondaryLabel), so it flips
  with the glass. Hard-coded white glyphs only on Clear glass over dark/dimmed media.

How Apple tints (to reproduce it): pick the tint in a perceptual space (OKLCH works),
keep **hue**, keep most **chroma**, and drive **lightness** from the luminance of the
frosted backdrop within a narrow band around the tint's own lightness. Result: the colour
reads as the same tint everywhere but brightens over light content and deepens over dark
content, like stained glass.

## 3. Layering: glass on glass

> "Always avoid glass on glass. … When placing elements on top of Liquid Glass, avoid
> applying the material to both layers. Instead, use fills, transparency, and vibrancy for
> the top elements to make them feel like a thin overlay that is part of the material."
> — WWDC25 "Meet Liquid Glass"

| Situation | Do |
|---|---|
| Button/icon inside a glass panel | plain shape + vibrant fill (see anatomy §2.5) or solid semantic colour. **No second glass.** |
| Several glass controls next to each other | one `GlassEffectContainer` (one render pass, they can merge/morph). Glass can't sample glass, the container solves it |
| Segmented control / selection pill inside a glass bar | a vibrant fill pill; Apple only "lifts" it into glass *while dragging* (liquid lens), then drops back to a fill |
| Menu or popover opening from a glass button | morph the button into the menu (same container + `glassEffectID`), don't stack a second glass on top |
| Unavoidable stacking (sheet over a toolbar, overlay over a tab bar) | upper glass must be **Regular** (thicker, more opaque); it must sample the *rendered* lower glass; never Clear over glass |
| Glass over a dimming layer (alert, sheet) | fine: the dim layer is content, not glass |

Liquid Rust implements the unavoidable case correctly: each z-layer samples a backdrop that
includes lower glass, and the backdrop is rebuilt **only** for the screen region where the
upper layer overlaps lower glass (see `liquid-rust.md`).

## 4. Where glass belongs

- **Only the functional layer** (navigation, toolbars, tab bars, floating buttons, sheets,
  menus, popovers, sliders/toggles while interacting). Never list rows, cards, body text,
  backgrounds [HIG].
- Exception: content-layer sliders/toggles **turn into** glass while the finger is on them.
- Use it sparingly; limit custom glass to the most important elements [HIG].
- In steady state (e.g. first launch) avoid intersections between content and glass —
  reposition content instead [Apple].
- Concentric corners: inner radius = outer radius − padding (`ConcentricRectangle`,
  `.containerConcentric`). Capsules for small controls; rounded rects with **continuous
  (squircle) corners** for big ones (iOS = Figma smoothing 60 %). This applies to glass
  *and* to fills sitting on glass (sub-buttons, album art, folder tiles). Use plain circular
  corners only for hardware-matched shapes (Dynamic Island) and tiny dots. Never mix
  circular and continuous corners within one cluster: the mismatch is visible in the rim
  highlight. See `anatomy.md` §2.7.

## 5. Legibility

- Regular glass self-adapts; Clear needs dimming.
- Scroll edge effects: soft fade under floating bars; **hard** style when pinned accessory
  views (column headers) sit under the toolbar [Apple].
- Text on glass: SF semibold or bolder for small sizes; use vibrant label colours, not
  fixed greys. Vibrancy on the web = `mix-blend-mode: plus-lighter` for light-on-dark
  (Apple kit: LINEAR_DODGE) and `plus-darker` / `multiply` for dark-on-light (kit: LINEAR_BURN).
- Big glass with lots of text (menus, sidebars, alerts) → Regular, larger is more opaque.

## 6. Accessibility (must be honoured by custom renderers)

| Setting | Native effect [Apple] | Custom renderer should |
|---|---|---|
| Reduce Transparency | frostier, obscures more content | frost ×3, face opacity → high, refraction ↓ |
| Increase Contrast | elements mostly black/white with a contrasting border | flatten tint, add 1 pt border in label colour |
| Reduce Motion | lower intensity, **no elastic** properties | springs critically damped (bounce 0), no stretch, no press scale |
| Transparency slider (27) | ultra clear → fully tinted | expose 0…1 knob mapped to frost + face opacity |
| Web | — | `prefers-reduced-transparency`, `prefers-contrast`, `prefers-reduced-motion` |

## 7. Motion rules (summary — details in `motion.md`)

- Everything is a **spring**, retargetable mid-flight, velocity preserved.
- Press: fast (≈0.25 s) spring up; release: bouncy (bounce ≈0.3) settle.
- Morph and layout changes: `.smooth` / `.snappy`.
- Drag of a lens/knob: follow tightly, stretch along velocity, jelly-settle on stop.
- Never animate glass with linear/ease curves or alpha crossfades.

## 8. Anti-patterns

| Don't | Because |
|---|---|
| Glass on every card/list row | competes with content, muddy hierarchy |
| Tint every button | nothing stands out |
| Solid colour fill on a glass button | breaks the material |
| Glass inside glass | clutter, double refraction; use fills/vibrancy |
| Clear glass over plain UI | illegible; Clear is for media only |
| Mixing Regular and Clear in one toolbar | inconsistent material |
| Big blur + white overlay | that's 2020 glassmorphism |
| Fixed white glyphs on Regular glass | won't flip when the glass goes light |
| Alpha-fade glass in/out | Apple materializes via lensing |
| Collision physics between glass blobs | Apple merges geometrically (SDF union), it never "bumps" |
