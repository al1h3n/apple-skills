# iOS / iPadOS / macOS NN (YEAR)

How to add a version:
1. Copy this file to `NN.md`. Fill every section. Tag every number `[Apple]` `[Kit]` `[M n]` `[D]`.
2. Move what the new OS replaced out of the current files (`anatomy.md`, `motion.md`, `components.md`, `effects.md`, `icons.md`, `design-rules.md`) into the previous version's file. Current files describe **only the newest OS**.
3. Update `SKILL.md` presets and the version line. Update `sources.md`.
4. Re-run the checks in `principles.md` §6: same eight situations measured, new column of numbers. Rules in `principles.md` change only if the evidence breaks them; say which evidence.

Measure before writing. The eight situations:

| # | Situation | Gives |
|---|---|---|
| 1 | App close → Home, app open | backdrop layers, τ per layer |
| 2 | Clear glass over a busy wallpaper, still + page swipe | frost σ, tone, refraction band, rim |
| 3 | Bar buttons: press, release, mode change, push | press growth, glow, fuse, bar timing |
| 4 | Banner arrive, rest, leave | origin, spring, material ramp |
| 5 | Switch tap, hold, drag, close-up | lens size, sequence, rim |
| 6 | Assistant: summon, listen, work, answer | shapes, τ, text swap |
| 7 | Menu from a button; sheet from a button | one-shape morph timing |
| 8 | Long capture: scroll under bars over light and dark, Control Center, camera, Photos | legibility flip, pages, materialize |

Kit: dump the new Figma libraries by key (every variant, nested components, light + dark), diff the glass stacks against the previous kit.

---

## 1. Apple's words [Apple]

## 2. Material, previous → NN [Kit]

| | previous | NN |
|---|---|---|
| Refraction / depth | | |
| Dispersion / splay | | |
| Light | | |
| Hairline, rims | | |
| Frost by role | | |
| Fills (small, panel, clear, tinted) | | |
| Shadow by size | | |
| States | | |

## 3. Motion, previous → NN [M]

| | previous | NN |
|---|---|---|
| Press growth, time to peak, light | | |
| Bar mode change | | |
| Emerge / recede τ | | |
| Banner | | |
| Knob lens | | |
| Drag | | |
| Backdrop | | |

## 4. Components, previous → NN [Kit]

## 5. Icons

## 6. APIs new in SDK NN

## 7. Reading

Which priority (`legibility > hierarchy > spatial continuity > calm > delight`) each change served.
