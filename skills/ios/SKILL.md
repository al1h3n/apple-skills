---
name: ios
description: >
  iPhone and iPad interface design, current as of iOS / iPadOS 27: system components with exact
  sizes from Apple's kit (tab bars, toolbars, sheets and detents, alerts, menus, lists, controls,
  keyboards), navigation patterns (tab bar, stack, split view, search placement, modality),
  Dynamic Type and accessibility sizes, gestures and touch feedback, system surfaces (widgets,
  Lock Screen, Control Center, notifications, Dynamic Island, Home Screen), safe areas and
  device sizes including foldables. Use for "design an iOS / iPadOS app or screen", "which iOS
  component", "tab bar or sidebar", "sheet detents", "Dynamic Type", "iOS touch targets",
  "widget design", "review iOS UI". Requires apple-ui (hierarchy, type, colour, copy, review)
  and liquid-glass (the material, motion numbers, native APIs).
---

# iOS and iPadOS

Only what is specific to iPhone and iPad. **Load with**:
- `apple-ui`: hierarchy, typography ramp, spacing, colour, copy, review checklist.
- `liquid-glass`: material, motion timings, per-component glass spec, SwiftUI / UIKit APIs.

Current OS: 27. Version notes: `references/versions/`.

## Pick the path

| Doing | Read |
|---|---|
| Choosing and sizing a component | `components.md` |
| App structure: tabs, stacks, split views, search, modals | `navigation.md` |
| Text sizes that follow the person's setting | `dynamic-type.md` |
| Touch, gestures, feedback | `gestures-and-motion.md` |
| Widgets, Lock Screen, Control Center, notifications, island | `widgets-and-system.md` |

## Core numbers [Kit]

| | iPhone (402 × 874 pt reference) | iPad |
|---|---|---|
| Status bar | 62 | 24–32 |
| Home indicator | 144 × 5, area 34 | area 21 |
| Top toolbar | 54; buttons 44 × 44 | 54; buttons 44 × 44 |
| Bottom toolbar | 84; buttons 48 × 48 | none |
| Tab bar | 95 (88 minimised); bar 62 tall | 44 tall capsule, top |
| Side margin | 16 | 20 |
| Floating panel inset | 6–8 | 8 |
| List row | 52 (68 tall) | same |
| Touch target | ≥ 44 × 44 | same |

## Rules specific to touch

1. **Thumb first.** Primary actions and search at the bottom. Top = context and rarely used actions.
2. **Bars float; content runs under them.** Never give a bar an opaque background. Use the scroll edge effect.
3. **Tab bar = top-level places only** (2–5). Actions go in toolbars. Never hide the tab bar on push.
4. **Sheets for a task, push for a hierarchy.** Partial sheet when the content behind still matters.
5. **Every gesture has a visible alternative.** Swipe actions also in a menu; edge-swipe back also a back button.
6. **Respond on touch-down, act on touch-up.** Feedback within one frame.
7. **Dynamic Type everywhere.** Layout reflows up to the accessibility sizes.
8. **Safe areas and corners.** Bottom panels follow the display's corner radius; nothing interactive under the home indicator.

## Files

`references/`: `components.md`, `navigation.md`, `dynamic-type.md`, `gestures-and-motion.md`, `widgets-and-system.md`, `versions/27.md`, `versions/26.md`, `versions/_template.md`.
