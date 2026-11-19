---
name: liquid-glass
description: >
  Apple Liquid Glass, current as of iOS / iPadOS / macOS 27 (26 kept as a version file): what the
  material is made of (Apple Figma kit layer stacks and GLASS parameters, Core Animation
  pipeline), design rules (Regular / Clear / Tinted, transparency slider, where colour goes,
  glass on glass, legibility flip, accessibility), per-component specs for iOS and macOS, motion
  measured frame by frame (launched springs, one-shape menu morph, knob lens, banners, assistant),
  app icons and Icon Composer, native SwiftUI / UIKit / AppKit APIs, GPU recreation (WGSL / WebGPU
  shader maths), the Liquid Rust engine (Rust + wgpu, native and wasm, mask glass in any
  silhouette) and its web core with Svelte 5 (Bits UI) and SolidJS (Kobalte) component kits. Use for any request about Liquid Glass, glass UI, "the iOS 26 / 27 glass look",
  glassEffect, Figma glass effect values, glass shaders, liquid merging of UI shapes, glass icons,
  or choosing glass vs fill vs tint.
---

# Liquid Glass

Lensing material for the **functional layer**: bars, buttons, menus, partial sheets floating over content. Bends light through a curved edge, tones the backdrop, lights its rim top and bottom, darkens a hairline outside, merges with neighbours in a group. **Not** glassmorphism (big blur + white fill).

Current version: **27**. Every file describes 27 only. Older: `references/versions/26.md`. New OS: `references/versions/_template.md`.

Tags on numbers: **[Apple]** Apple text. **[Kit]** Apple Figma kit. **[M1]…[M8]** measured on recording n. **[RE]** reverse-engineered. **[D]** derived. Detail: `references/sources.md`.

## Pick the path

| Building | Do | Read |
|---|---|---|
| Native Apple app | system API. Never hand-roll shaders there | `native-apple.md` |
| Svelte or SolidJS web app | `bits-ui-liquid-glass` / `solid-liquid-glass` (engine + CSS renderer, same components) | `bits-ui.md` |
| Other web | `liquid-rust-web`: `createLayer` + `glass`, or the engine alone | `webgpu.md` |
| Rust, any GPU / UI stack | Liquid Rust crate | `liquid-rust.md` |
| Other engine (Unity, Godot, Metal, GL) | port `assets/glass.wgsl` | `shader-math.md` |
| Figma / design file | kit layer recipe | `anatomy.md` §2–4 |
| A component | sizes, radii, glass or not | `components.md` |
| Motion | springs, timings | `motion.md` |
| System moments (backdrop zoom, banner, assistant) | recipes | `effects.md` |
| App icon | layers, finish, generator (LIGMA) | `icons.md` |
| Review / "is this right?" | rules + checklists | `design-rules.md`, `anatomy.md` §10 |
| New design, no precedent | priorities, rulings | `principles.md` |

Always read `design-rules.md` before choosing colours or stacking glass.

## Priority

**legibility > hierarchy > spatial continuity > calm > delight.** Conflict → higher wins. (`principles.md`)

## Presets [Kit]

| Preset | Use | frost / refraction / depth / dispersion / splay / light | Tone |
|---|---|---|---|
| **Regular small** | buttons, bars, tab bars, search | 6 / 0.7 / 30 / 0.2 / 0.2 / 0.25 | light translucent face; adapts light↔dark to backdrop |
| **Regular panel** | menus, sheets, alerts, pickers | 16 / same lens | 70 % white Lighten (dark: 70–90 % `#1A1A1A`); follows scheme |
| **Clear** | controls over media and wallpaper | 6 / same lens / light 0.40 | almost none; white content; host dims 35 % when bright |
| **Tinted** | the one primary action | Regular-small lens | 94 % white backing + accent Linear Burn = near-opaque; label by contrast |
| **Lens** | knob while pressed | 0 / 0.8 / 7 / 0 / 0 / 0 | 9 % white |
| Identity | transition endpoint | none | nothing |

One lens for everything. Size changes frost + tone, never refraction. Shadow: controls 2 % (edge = hairline), panels 25 % blur 48.

User setting: transparency slider, ultra clear → fully tinted. Expose one 0…1 knob in custom renderers.

## Non-negotiables

1. **Bevel refraction**: band ≈ 30 pt along the edge (less on small shapes), centre flat and colour-clean. Faint dispersion inside the band only.
2. **Tone below the lens**, applied in sRGB like Figma fills. Not a white overlay on top.
3. **Rim light top + bottom** (additive, ≈ 10 pt soft) and a **dark burn hairline** 0.5 pt outside, stronger at the sides. No white border. No single top-left blob.
4. **One glass per stack.** On glass: fills + vibrancy (burn in light, dodge in dark). Selection = burn capsule.
5. **Merge by SDF smooth union** inside one container (`k` = spacing). Never collision physics. **Press does not fuse neighbours.**
6. **Launched motion**: finger-sent things start at full speed, land exponentially (emerge τ 72 ms, recede τ 48 ms). No ease-in. Bounce ≤ 0.2 and only on arrivals with content.
7. **One shape, not two**: a menu / sheet is its button growing (blob → dome → corners tighten), content inside from frame 1. Button absent while open.
8. **Press**: +5 pt, cap 15 %, peak in 50 ms, lights from the finger, glyph stays readable; glow-out τ 90 ms. Knob → lens ×1.6 while held. Drag: lift, no stretch.
9. **Materialize by lensing**, never alpha. Content soft while its surface moves, sharp on landing.
10. **Legibility per element on CPU**: small Regular adapts (to light instantly, to dark after 0.1 s dwell, hysteresis); panels follow the scheme; Clear never flips; tinted label by contrast. Host reads the tone, never guesses.
11. **Inactive window → glass off** (flat fill). Full-height sheets opaque.
12. **Continuous corners** on panels and on fills sitting on them; capsules for controls; concentric radii; per-corner radii where the display corner dictates.
13. **Accessibility**: Reduce Transparency, Increase Contrast, Reduce Motion, the slider.
14. **Render on demand**: idle = 0 frames.

## Colour in one glance

- Primary action → tint the glass background. One per screen. Shows up when usable.
- Toggle state in a panel → opaque system-colour circle, white glyph. Not glass.
- Meaning of a value → colour the glyph / value text.
- Bars over colourful content → monochrome glyphs that flip with the glass.
- Brand → content layer.

## Quick start, native

```swift
GlassEffectContainer(spacing: 16) {
    HStack(spacing: 12) {
        Button("Edit") {}.buttonStyle(.glass)
        Button("Done") {}.buttonStyle(.glassProminent)
    }
}
```

Engine and web quick starts: `liquid-rust.md`, `bits-ui.md`, `webgpu.md`.

## Files

| File | Holds |
|---|---|
| `references/principles.md` | priority order, seven rules, rulings on conflicting evidence, Disney 12 → UI, five motion rules |
| `references/anatomy.md` | layer stack, glass recipes, material stacks, states, corners, compositor, look checklist |
| `references/design-rules.md` | variants, colour, layering, where glass belongs, legibility ruling, accessibility, anti-patterns |
| `references/components.md` | per-control spec, iOS / iPadOS / macOS |
| `references/motion.md` | spring maths (copy-ready), launched motion, choreography table, pacing, Reduce Motion |
| `references/effects.md` | backdrop depth, banner, assistant island, answer card, Control Center, camera |
| `references/icons.md` | Icon Composer model, glass finish, generator recipe |
| `references/versions/27.md`, `references/versions/26.md`, `references/versions/_template.md` | what changed per OS; how to add the next |
| `references/native-apple.md` | SwiftUI, UIKit, AppKit |
| `references/shader-math.md` | full algorithm, calibration, bug table |
| `references/webgpu.md` | browser paths, backdrop capture, HTML-in-Canvas, CSS renderer |
| `references/liquid-rust.md` | engine API, kit components, recipes |
| `references/bits-ui.md` | Svelte and SolidJS kits over Bits UI and Kobalte |
| `references/sources.md` | what each claim rests on |
| `assets/glass.wgsl`, `assets/pyramid.wgsl` | tested shaders, synced with the engine |

Pairs with `apple-ui` (hierarchy, type, layout, copy, review) and `ios` (iOS components, navigation).
