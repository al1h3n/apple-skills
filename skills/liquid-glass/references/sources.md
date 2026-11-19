# Sources and confidence

Assembled November 2026 (iOS / macOS 27.2 shipping, 27.3 in beta). Tags used across the skill:

| Tag | Means | Trust |
|---|---|---|
| **[Apple]** | Apple's page, session or keynote | rule: binding. Number: until a newer kit disagrees |
| **[Kit]** | Apple's Figma libraries, read node by node through the plugin API | highest for numbers |
| **[M1]…[M8]** | measured frame by frame on recording n | timing exact to one frame (17–33 ms); sizes ±5 %; colours directional |
| **[RE]** | reverse-engineered from Apple's private pipeline | structure reliable, values per-OS |
| **[D]** | derived here | tune by eye |

## Apple [Apple]

- HIG *Materials* (last change September 9, 2025): Regular vs Clear, three conditions for Clear, 35 % dimming, no glass in content layer, scroll edge effects, standard materials for content.
- HIG *Color*, "Liquid Glass color" (December 16, 2025): colour the background for the primary action, colour sparingly, monochrome bars over colourful content.
- HIG *App icons* (June 8, 2026; October 28, 2026): layers, formats, appearances, what not to bake in.
- WWDC25 219 "Meet Liquid Glass": lensing, materialize, adaptive shadow, light/dark flip, illumination from the touch point, no glass on glass, tint mapping, accessibility.
- WWDC25 361, WWDC26 group lab 8012: Icon Composer (groups, specular, blur, refraction, translucency, shadow, mono, pinned clear glass).
- WWDC26 keynote + Platforms State of the Union + 269 "What's new in SwiftUI": better diffusion, darkened edge, brighter specular, transparency slider, uniform toolbar on scroll, `appearsActive`, toolbar additions, icon refraction.
- WWDC23 "Animate with springs": duration / bounce model.
- Developer docs, SDK 27: `Glass`, `glassEffect`, `GlassEffectContainer`, `glassEffectID`, `glassEffectUnion`, `GlassEffectTransition`, `GlassButtonStyle`, `UIGlassEffect`, `UIGlassContainerEffect`, `UIButton.Configuration.glass()` family, `UICornerConfiguration`, `NSGlassEffectView`, `NSGlassEffectContainerView`, `ToolbarOverflowMenu`.
- Press on betas: MacRumors (10 and 16 June 2026), 9to5Mac (15 June 2026), Beta Profiles (27.3 beta 1: wallpaper zoom-out restored).

## Figma kits [Kit]

Community libraries by Apple, every component imported by key, every variant and nested component serialised (fills, blend modes, effects incl. `GLASS`, variables, text, auto-layout), light + dark renders:

| Library | Components read |
|---|---|
| iOS and iPadOS 27 | 198 |
| macOS 27 | 115 |
| iOS and iPadOS 26 | 142 |
| macOS 26 | 56 |

Basis of `anatomy.md`, `components.md`, `versions/26.md`, `versions/27.md`.

## Recordings [M1]…[M8]

Every frame extracted; motion events found by frame-difference energy; edges tracked per frame; curves fitted (exponential τ, spring duration/bounce).

| # | Recording | Compares | Gives |
|---|---|---|---|
| M1 | background effect and zoom, 822 frames | iOS 27.2 vs 27.3 beta | backdrop layers on app close/open |
| M2 | bend and blur, 1392 | 26.6.1 vs 27 | Clear glass frost, tone, refraction band, rim |
| M3 | bar animations, 1238 | 26.5 vs 27 beta | press, release, bar mode change, rest look of Regular |
| M4 | notification, 777 | 27 beta 5 vs beta 6 | banner arrival, rest, dismissal |
| M5 | switch, 1549 | 26.5 vs 27 beta 2 | knob lens geometry and sequence |
| M6 | assistant, 1408 | 27 | island → drop → pill → bar, text swap, rim light |
| M7 | Apple's assistant film, 1788 + 666 | 27 | answer card, menu from a button, Lock Screen notification |
| M8 | hand-held capture of an iPhone, 15,419 | 27 | legibility flip, Control Center, sheets, camera, Photos |

**[M9]** macOS 27 screen recordings, 1920 × 1080 at 60 fps, 1 px ≈ 1 pt: Mail and Photos
windows, the AirPods equalizer and Listening Mode sliders, Siri, Visual Intelligence, a
28,500-frame walk through Settings, Dock, traffic lights and Spotlight, and 71 app icons
side by side in 26 and 27. Notes: the liquid-rust repository, docs/research (macos-27 and
icons-macos-26-vs-27).

Limits: 60 fps capture of a 120 Hz display (fast things show doubled); M1/M4/M5/M6 are 30 fps content; M8 is a camera filming a screen (levels relative, timing exact); icon references are small web images.

## Reverse engineering [RE]

- **ShatteredGlass** (AlexStrNik): macOS 26 Core Animation tree. `CABackdropLayer` (scale 0.25) + `glassBackground` CAFilter + `CASDFLayer` union + two opposite `CASDFGlassHighlightEffect` with `vibrantColorMatrix`. Values in `anatomy.md` §8.
- **LiquidGlassKit** (DnV1eX): Metal reimplementation of `_UILiquidLensView`: refraction from SDF depth, glare as an LCH-boosted backdrop, velocity-driven lens scale.

## Techniques (ideas checked, code not copied)

- kube.io "Liquid Glass in the browser": surface profiles, Snell displacement, SVG displacement maps.
- figma_glass (Flutter): Figma parameter semantics.
- Inigo Quilez: rounded-box SDF, smooth minimum with gradients.
- Jimenez 2014 (Call of Duty: Advanced Warfare): 13-tap downsample.
- Chrome: HTML-in-Canvas, stable in Chrome 157 (`developer.chrome.com/blog/html-in-canvas`).

## What is [D]

- σ = frost / 2; lit-line width; size → thickness curve; adaptive thresholds (confirm against M8 when retuning).
- Hover values. Reduce Motion replacements. Transparency-slider mapping.
- Icon generator widths and strengths (`icons.md` §4).
