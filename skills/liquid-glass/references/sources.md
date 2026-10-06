# Sources and confidence

Assembled October 2026 (iOS/macOS 27 shipped). Tags used across the skill:
**[Apple]**, **[Kit]**, **[RE]**, **[Derived]**.

## Official Apple [Apple]

- Developer docs (verified through the doc JSON, SDK 27):
  - `Glass` — `regular`, `clear`, `identity`, `tint(_:)`, `interactive(_:)`
  - `glassEffect(_:in:)`, `GlassEffectContainer(spacing:)`, `glassEffectID(_:in:)`,
    `glassEffectUnion(id:namespace:)`, `GlassEffectTransition` (`matchedGeometry`, `materialize`, `identity`)
  - `GlassButtonStyle` / `.glass` / `.glass(_:)` (26.1) / `.glassProminent`
  - UIKit: `UIGlassEffect(style:)` (`.regular` / `.clear`, `isInteractive`, `tintColor`),
    `UIGlassContainerEffect.spacing`, `UIButton.Configuration.glass()` / `.prominentGlass()` /
    `.clearGlass()` / `.prominentClearGlass()`, `UICornerConfiguration.capsule()`
  - AppKit: `NSGlassEffectView` (`style`, `tintColor`, `cornerRadius`, `effectIsInteractive`),
    `NSGlassEffectContainerView`, `NSButton.BezelStyle.glass`
  - `ToolbarOverflowMenu` (27.0)
- Technology overview "Adopting Liquid Glass" and article "Applying Liquid Glass to custom views".
- HIG *Materials* (updated Sep 9 2025): regular vs clear, the three conditions for clear,
  35 % dimming, no glass in the content layer.
- HIG *Color*, section "Liquid Glass color" (updated Dec 16 2025): colour the background for
  primary actions, use colour sparingly, monochrome bars over colourful content.
- WWDC25 219 "Meet Liquid Glass" (full transcript read): lensing, materialize, adaptive
  shadows, flip light/dark, highlights, illumination from the touch point, no glass on glass,
  regular vs clear, tint mapping, accessibility.
- WWDC26 102 Platforms State of the Union and 269 "What's new in SwiftUI" (transcripts):
  better diffusion, the slider from "ultra clear to fully tinted", sidebars to the edge,
  pointer-interactive glass on macOS, `appearsActive`, new toolbar APIs, icon refraction,
  Xcode 27 adoption.
- iOS 26.1 "Clear / Tinted" Liquid Glass setting (Engadget, TechCrunch, Tom's Guide, Oct 2025).
- Spring model: WWDC23 "Animate with springs" (duration/bounce → stiffness/damping).

## Apple Figma kit [Kit]

User's file "Template for UI" → frame "iOS + IpadOS". Every effect stack was read through the
Figma Plugin API: GLASS, INNER_SHADOW, DROP_SHADOW, fills and blend modes. Renders at 4× were
compared against Liquid Rust pixel profiles. This is the most precise public description of
how Apple builds the material, and it is the basis of `anatomy.md` §2 and of all presets.

## Reverse engineering [RE]

- **ShatteredGlass** (AlexStrNik): the macOS 26 Core Animation layer tree. `CABackdropLayer`
  (scale 0.25) + `glassBackground` CAFilter (blur, inner/outer refraction, face colour matrix,
  luma-aware shadow, bleed, HDR) + `CASDFLayer` union + two opposite `CASDFGlassHighlightEffect`s
  with `vibrantColorMatrix`. The parameter values are in `anatomy.md` §3.
- **LiquidGlassKit** (DnV1eX): a Metal reimplementation of `_UILiquidLensView`. Refraction from
  SDF depth, glare taken as an LCH-boosted backdrop, a velocity-driven lens scale.

## Community techniques (ideas checked, code not copied)

- kube.io "Liquid Glass in the browser": surface profiles (the squircle), Snell displacement,
  SVG displacement maps.
- figma_glass (Flutter): Figma parameter semantics (depth = bevel px, splay = falloff,
  light peak and opposite peak).
- Inigo Quilez: rounded-box SDF, smooth minimum with gradients.
- Jimenez 2014 (Call of Duty: Advanced Warfare): the 13-tap downsample.
- Chrome blog: HTML-in-Canvas origin-trial API shape (Chrome 155+).
- The user's earlier `liquid-claude` project: reviewed for its failure modes only (Euler
  physics, collisions, per-pixel full-scene loops, `dFdx` normals).

## What is [Derived] (tune by eye)

- The regular dark tone, the size → thickness curve (τ 44–320 pt), and the adaptive thresholds.
- Press, glow and stretch amplitudes.
- The transparency-slider mapping.
- The lit-line width of 0.25 pt × (1 + splay): fitted to the kit render.
