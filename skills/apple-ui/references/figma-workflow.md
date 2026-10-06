# Building it in Figma

## 1. Before editing

- Back up first: clone everything into a locked section ("BACKUP · <date>") below the work.
- Inventory: pages, components, styles, variables; match existing naming.

## 2. Tokens

- Paint styles: `Glass/Regular fill`, `On glass/Fill`, `On glass/Selected`, `On glass/Divider`,
  `Label/Primary`, `Label/Secondary`, `AI/Outline`, plus `Light/*` and `Solid/*` twins.
- Effect styles: `Glass/Regular`, `Glass/Clear`, `Solid/Shadow`.
- Text styles: `App/<HIG role>` and `Landing/<step>` (see typography).
- Variables with modes are the ideal theme switch, but the free Figma plan allows **one mode
  per collection**. Use **component properties** instead: build each part/screen as a
  component with variants `Theme = Dark | Light` × `Liquid Glass = On | Off` and restyle the
  copies by swapping style ids. Upgrading later = move styles to variables.

## 3. Structure and names

- Auto-layout for anything with related children; absolute only for overlays.
- Name by meaning: `Row · Liquid Glass`, `Directions · primary action (solid colour)`,
  `Search field`, `Tourist card · smoked glass`.
- Keep the same layer names across variants so Smart Animate morphs them.
- Notes: one group per note = yellow card + leader line + dot on the target edge
  ("Note · <topic>"), never loose arrows and texts.

## 4. Prototypes

- Variants + CHANGE_TO + SMART_ANIMATE with CUSTOM_SPRING (stiffness/damping from the motion
  table). Self-playing demos: AFTER_TIMEOUT on variants. Screens: NAVIGATE between top-level
  frames.
- Use real kit components for system UI (status bar, home indicator, toggles) instead of
  drawing them.

## 5. Automating with the Plugin API

- Small, idempotent steps (guard on a node name each step creates); return ids.
- Real plugins lack MCP helpers (`createAutoLayout`, `query`); `resize()` resets hug to fixed;
  `INNER_SHADOW` has no `showShadowBehindNode`; springs take mass/stiffness/damping only;
  read `mainComponent` with `getMainComponentAsync()` in dynamic-page mode; load other pages
  with `loadAsync()`.
