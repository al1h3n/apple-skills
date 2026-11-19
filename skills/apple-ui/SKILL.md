---
name: apple-ui
description: >
  Apple-grade product UI on any platform, and the marketing around it. Shared craft: focus and
  hierarchy (one job per screen, one primary action, progressive disclosure), typography (HIG
  ramps for iOS and macOS, SF licensing and free alternatives, modular scales), layout and
  spacing (8 pt rhythm, targets, concentric continuous corners), colour discipline (semantic
  colours, one accent, light / dark / increased contrast), motion principles (launched springs,
  one shape, when not to animate), copywriting, loading / empty / error states, accessibility,
  landing pages that convert, AI assistant UI, a premium-UI review checklist, Figma workflow, and
  macOS specifics (windows, toolbars, sidebars, menus, pointer). Use for "make this UI better /
  more Apple / premium", "design a screen / landing page / settings / AI chat / Mac app",
  "review this UI", type scales, microcopy. Pairs with liquid-glass (the material) and ios
  (iPhone / iPad components and navigation).
---

# Apple UI

Restraint. One job per screen. Content first, chrome recedes. One accent. Real type hierarchy. Motion that explains change. Marketing: same rules + show the product working.

Scope: craft shared by every platform + macOS. iPhone / iPad specifics → `ios`. Glass material → `liquid-glass`. Current OS: 27; version notes in `references/versions/`.

## Pick the path

| Doing | Read |
|---|---|
| What goes on a screen, what is primary, what hides | `focus-and-hierarchy.md` |
| Fonts, sizes, weights, scales | `typography.md` |
| Spacing, grids, radii, targets, scroll | `layout-and-spacing.md` |
| Colours, dark / light, contrast, glass vs solid | `color-and-materials.md` |
| Why and how things move | `motion-principles.md` (numbers: liquid-glass `motion.md`) |
| Labels, buttons, errors, empty states | `copywriting.md` |
| Landing page, hero, demo, CTA | `marketing-pages.md` |
| AI assistant (quick card, window, voice) | `ai-assistant-ui.md` |
| Mac app: windows, toolbars, sidebars, menus | `macos.md` |
| Reviewing finished work | `review-checklist.md` |
| Building in Figma | `figma-workflow.md` |

## Ten rules

1. **One job per screen or panel.** Needs a scrollbar to explain itself → split.
2. **One primary action, one accent.** Solid colour = the thing to do next. Rest = quiet fills.
3. **Content first.** Controls float and stay small. Remove boxes round things already grouped.
4. **Progressive disclosure.** Top 5–6 choices, then "More…". Advanced, credits, long text fold away.
5. **Real type hierarchy.** A defined ramp, two or three weights. No in-between sizes.
6. **Group by space, not lines.** 8 pt rhythm. Hairlines only between rows of one group, inset to text, light.
7. **Concentric continuous corners.** Inner radius = outer − padding. Capsules for small controls.
8. **Every state designed.** Rest, hover, pressed, focus, disabled, loading, empty, error, offline.
9. **Motion explains change.** Launched springs; things grow out of what spawned them; nothing from nowhere. Repeated hundreds of times a day → barely animate.
10. **Words are UI.** Short, concrete, benefit first. One line; the second line belongs in "Learn more".

Tie-break order (same as liquid-glass): **legibility > hierarchy > spatial continuity > calm > delight.**

## Quick numbers

| | iOS / iPadOS | macOS |
|---|---|---|
| Body | 17/22 | 13/16 |
| Control height | 44–50 (text buttons 28 / 34 / 50) | 24, toolbar XL 36 |
| Target | ≥ 44 × 44 touch | ≥ 24 pointer, 28 preferred |
| Margins | 16–20 | 20; window content 20 |
| Row | 44–52 | 24 (sidebar 24 / 32 / 36), table 20 |
| Panel radius | 34 (menu, alert), sheet 38 | menu 12, popover 20, alert 34, window 16 |

- Spacing steps 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64.
- Contrast ≥ 4.5 : 1 text under 18 pt; ≥ 3 : 1 large text, icons.
- Line length 45–75 characters; marketing body 17–21 px at 1.4–1.5.
- Motion: open τ 72 ms, close τ 48 ms, press peak 50 ms; nothing repeated above 0.2 s.

## Fonts in a shipped product

- **SF Pro / SF Compact / SF Mono / New York**: licensed for designing and mocking up UIs for Apple platforms only. Never embed in web, Windows or Android builds. Web: system stack `system-ui, -apple-system, "Segoe UI", Roboto, sans-serif` (Apple devices draw SF themselves; nothing distributed).
- Free to ship (SIL OFL): Inter, Inter Display, Geist, Manrope, IBM Plex Sans. Serif: Instrument Serif, Newsreader, Source Serif 4, Fraunces. Mono: JetBrains Mono, Geist Mono.

## Workflow

1. Write the screen's job + primary action in one sentence before drawing.
2. List states and the content visible at rest.
3. Ramp, spacing grid, one accent → tokens first.
4. Build top-down. Name layers by meaning.
5. Squint test (primary action wins?), 3-second test (what is this screen for?), `review-checklist.md`. Fix. Show over real content at real device size.

## Files

`references/`: `focus-and-hierarchy.md`, `typography.md`, `layout-and-spacing.md`, `color-and-materials.md`, `motion-principles.md`, `copywriting.md`, `marketing-pages.md`, `ai-assistant-ui.md`, `macos.md`, `review-checklist.md`, `figma-workflow.md`, `versions/27.md`, `versions/26.md`.
