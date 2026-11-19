# Building it in Figma

## 1. Before editing

- Back up: clone into a locked section ("BACKUP · <date>") below the work.
- Inventory pages, components, styles, variables. Match existing naming.

## 2. Use Apple's kits, not drawings

Community libraries by Apple: "iOS and iPadOS 27", "macOS 27" (older: 26, plus watchOS, visionOS). Assets → Libraries → enable. Use real components for system UI (status bar, home indicator, switches, bars, keyboards).

Glass in the kits = two frames of the same shape: `Fill + Shadow` under `Glass Effect` (liquid-glass `anatomy.md` §2). Copy that pair for custom glass; bind frost to `Liquid Glass/Frost - Regular | Medium | Large`.

## 3. Tokens

- Paint styles: `Glass/Regular fill`, `On glass/Fill`, `On glass/Selected`, `On glass/Divider`, `Label/Primary`, `Label/Secondary`, plus `Light/*`, `Solid/*` twins.
- Effect styles: `Glass/Regular small`, `Glass/Regular panel`, `Glass/Clear`, `Solid/Shadow`.
- Text styles: `App/<HIG role>`, `Landing/<step>`.
- Variables with modes = ideal theme switch. Free plan: **one mode per collection** → use component properties instead: variants `Theme = Dark | Light` × `Liquid Glass = On | Off`, restyle copies by swapping style ids.

## 4. Structure and names

- Auto-layout for anything with related children. Absolute only for overlays.
- Name by meaning: `Row · Liquid Glass`, `Directions · primary action`.
- Same layer names across variants → Smart Animate morphs them.
- Notes: one group per note = card + leader line + dot on the target edge.

## 5. Prototypes

- Variants + CHANGE_TO + SMART_ANIMATE with CUSTOM_SPRING (values: `motion-principles.md` §3). Self-playing demos: AFTER_TIMEOUT. Screens: NAVIGATE between top-level frames.
- Figma springs start from rest: a launched motion can only be approximated (use the recede values for anything a tap opens).

## 6. Plugin API (automation, reading kits)

- Small idempotent steps (guard on a node name each step creates); return ids.
- `documentAccess: "dynamic-page"`: `getMainComponentAsync()`, `page.loadAsync()`, `getNodeByIdAsync()`.
- Library components by key: `figma.importComponentByKeyAsync(key)`, `importComponentSetByKeyAsync(key)`. Listing library variables needs `"permissions": ["teamlibrary"]`. `figma.fileKey` needs `"enablePrivatePluginApi": true` (development plugins only).
- Network from the UI iframe: declare `networkAccess.allowedDomains` (and `devAllowedDomains` for localhost).
- `GLASS` effect fields: `radius` (frost), `refraction`, `depth`, `dispersion`, `splay`, `lightAngle`, `lightIntensity`.
- `resize()` resets hug to fixed. `INNER_SHADOW` has no `showShadowBehindNode`. Springs take mass / stiffness / damping only.
- First `exportAsync` after setting an image fill can render blank: export once at tiny scale first.
- Remote nodes can disappear between calls: guard every read.
- Window in the background → timers throttle to about once a minute. Keep Figma in front for long runs.
