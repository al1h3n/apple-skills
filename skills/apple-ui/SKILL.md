---
name: apple-ui
description: >
  Apple-grade product UI and the marketing around it: focus and hierarchy (one job per screen,
  one primary action, progressive disclosure), Apple HIG typography (Dynamic Type ramp, SF
  licensing and free alternatives, modular scales for marketing pages), layout and spacing
  (8 pt grid, 44 pt targets, concentric squircle corners, sheets, inset grouped lists),
  colour discipline (semantic system colours, one accent, dark/light pairs), motion
  (springs, choreography, Reduce Motion), Apple-style copywriting (short, benefit first,
  verbs on buttons), loading/empty/error states, accessibility, and landing pages that
  convert (promise headline, touchable product demo, numbered feature grid, proof strip,
  product shot, repeated CTA). Use for "make this UI better / more Apple / catchy",
  "design a screen / landing page / settings / AI chat", "review this UI", type scales,
  microcopy, or turning a feature into a page people want. Pairs with liquid-glass for
  the glass material itself.
---

# Apple UI

Apple-quality interfaces come from **restraint**: one clear job per screen, content first,
chrome that recedes, a single accent, real type hierarchy, and motion that explains what
changed. Marketing pages follow the same rules, with one more: show the product working.

## Pick the path

| You are doing | Read first |
|---|---|
| Deciding what goes on a screen, what is primary, what hides | `references/focus-and-hierarchy.md` |
| Fonts, sizes, weights, line height, a type scale | `references/typography.md` |
| Spacing, grids, corner radii, sheets, lists, touch targets | `references/layout-and-spacing.md` |
| Colours, dark/light, tint, contrast, glass vs solid | `references/color-and-materials.md` (+ liquid-glass) |
| Springs, transitions, hover/press, reduced motion | `references/motion.md` |
| Labels, buttons, descriptions, errors, empty states | `references/copywriting.md` |
| A landing page, hero, demo, feature grid, CTA | `references/marketing-pages.md` |
| AI assistant UI (quick card, full window, voice, sessions) | `references/ai-assistant-ui.md` |
| Reviewing finished work | `references/review-checklist.md` |
| Building it in Figma (styles, themes, components, notes) | `references/figma-workflow.md` |

## The ten rules

1. **One job per screen or panel.** If a panel needs a scrollbar to explain itself, split it
   (Settings / City life / Layers / Sources became four panels, not one long one).
2. **One primary action, one accent.** Solid colour marks the single thing to do next
   (Directions). Everything else is a quiet fill. When every element is tinted, nothing is.
3. **Content first, chrome recedes.** Controls float over content and stay small; the map,
   photo or text is the hero. Remove boxes around things that are already grouped.
4. **Progressive disclosure.** Show the top 5–6 choices, then "More…". Credits, advanced
   options, long explanations fold away until asked.
5. **Real type hierarchy.** Use a defined ramp (Apple HIG for app UI, a modular scale for
   marketing) and two or three weights. Never invent in-between sizes.
6. **Grouping by space, not lines.** Inset grouped lists, 8 pt rhythm, hairlines only between
   rows of one group, inset to the text.
7. **Concentric continuous corners.** Inner radius = outer radius − padding; squircles
   (Figma smoothing 60 %) for rounded rectangles; circles/capsules for small controls.
8. **Every state designed.** Default, hover, pressed, focus, disabled, loading (skeleton
   pills, never spinners for content), empty, error, offline.
9. **Motion explains change.** Springs (duration + bounce), things grow out of what spawned
   them, nothing fades in from nowhere; Reduce Motion removes bounce, not meaning.
10. **Words are UI.** Short, concrete, benefit first. A description is one line; the second
    line belongs in "Learn more".

## Quick numbers

- iOS body 17/22, footnote 13/18, title 2 22/28, large title 34/41 (full table in typography).
- Touch target ≥ 44 × 44 pt; pointer target ≥ 28 pt; spacing steps 4, 8, 12, 16, 24, 32.
- Margins: iPhone 16–20 pt, iPad 20–24, desktop panels 24.
- Contrast ≥ 4.5 : 1 for text under 18 pt, ≥ 3 : 1 for large text and icons.
- Line length 45–75 characters; marketing body 17–21 px at 1.4–1.5 line height.
- Springs: smooth 0.5 s / bounce 0, snappy 0.5 / 0.15, bouncy 0.5 / 0.3.

## Fonts in a shipped product (licensing)

- **SF Pro / SF Compact / New York** may only be used to design and mock up interfaces for
  Apple platforms; they may not be embedded or redistributed in web, Windows or Android
  products. In a web app use the **system stack** (`system-ui, -apple-system, "Segoe UI",
  Roboto, sans-serif`): Apple devices render SF themselves, nothing is distributed.
- Free (SIL OFL) alternatives you can ship everywhere: **Inter** / Inter Display, **Geist**,
  **Manrope**; serif display: **Instrument Serif**, **Newsreader**, **Source Serif 4**,
  **Fraunces**. Claude.ai's look comes from commercial faces (Tiempos-like serif +
  Styrene-like sans); Instrument Serif / Newsreader + Inter get close for free.

## Workflow

1. Write the screen's one job and its primary action in a sentence before drawing.
2. List the states (see review checklist) and the content that must be visible at rest.
3. Choose the ramp, spacing grid and the one accent; set them as styles/tokens first.
4. Build top-down; group with auto-layout; name layers by meaning ("Row · Liquid Glass").
5. Squint test (does the primary action win?), 3-second test (can someone say what this
   screen is for?), and the review checklist. Fix, then show it in context (over real
   content, on the real device size).
