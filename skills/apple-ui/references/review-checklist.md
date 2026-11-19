# Review checklist

Run on every screen before calling it done. "Plausible" fails; each box is yes or fix.

## Focus
- [ ] Job fits one sentence. One primary action, first in a squint test.
- [ ] No shape exists only to decorate.
- [ ] Long lists use "More…". Long text folded. Credits folded except required ones.

## Type
- [ ] Every size on the ramp. ≤ 3 weights. One UI family.
- [ ] Secondary text in the secondary colour. Line length 45–75.
- [ ] Body leading ≥ 1.29; reading text 1.4–1.5.
- [ ] No widows in display headlines. Truncation with an ellipsis.

## Layout
- [ ] 8 pt rhythm. Same margins everywhere. Few alignment edges.
- [ ] Targets ≥ 44 touch / 24 pointer. Primary action in thumb reach.
- [ ] Concentric continuous corners. Hairlines light, inset to text.
- [ ] No visible scrollbars in panels.

## Colour and material
- [ ] One accent. Semantic colours consistent. Contrast ≥ 4.5 : 1 against every tone the surface takes.
- [ ] Light and dark designed. Glass on and off designed.
- [ ] Glass only on the functional layer. Never glass on glass. Clear only over media.
- [ ] Tinted control: one, appears when usable, label by contrast.

## States
- [ ] Rest, hover, pressed, focus-visible, disabled.
- [ ] Loading (skeletons shaped like the result), empty (what appears + one action), error (what happened + what to do), offline.
- [ ] Longest name, biggest number, no image, darkest and lightest image, small and large Dynamic Type, RTL if shipped.
- [ ] Inactive window (desktop).

## Motion
- [ ] Every surface grows from its source and returns to it. One shape, not two.
- [ ] Nothing a finger sent eases in. Nothing repeated runs above 0.2 s.
- [ ] No bounce on controls or dismissals.
- [ ] No snap: no state change without a transition, no jump-then-animate, no element vanishing in one frame.
- [ ] Interrupt mid-flight (tap again, reverse, resize) → continues smoothly.
- [ ] Reduce Motion version exists. Nothing loops faster than every 3 s.
- [ ] Idle screen renders 0 frames.

## Words
- [ ] Titles ≤ 4 words. Descriptions one line. Buttons = verb + result.
- [ ] No jargon, no source names in descriptions, no redundant labels.

## Legal
- [ ] Shipped fonts licensed for the platform (no embedded SF outside Apple platforms).
- [ ] Required attributions visible.
- [ ] Real owner name. No placeholder handles.

---

## Premium-UI mistakes (15) and their fixes

| # | Mistake | Fix |
|---|---|---|
| 1 | Icons over images lose contrast on some backgrounds | give the icon its own backing (material, glass, scrim); test on the darkest and lightest image |
| 2 | Cluttered or inconsistent product imagery | one photographic system: same background, light, crop, scale |
| 3 | Inconsistent grid margins | one margin value per breakpoint, every block on it |
| 4 | Saturated palette overpowering the product | neutral chrome; saturation for the product and the one action |
| 5 | Too many font styles | one family, ramp sizes, ≤ 3 weights |
| 6 | Cramped letter spacing in labels; icon repeating the badge text | normal tracking; one signal per badge |
| 7 | Titles that do not lead (size and weight too close to body) | step up size **and** weight |
| 8 | Tight paragraph leading | 1.4–1.5 for reading text |
| 9 | Trust signals far from what they vouch for | rating + count directly under the title |
| 10 | Feature icons mixing filled / outline and random colours | one family, one weight, one colour logic |
| 11 | Dark divider lines | separator colour, hairline, inset; or spacing instead |
| 12 | Excess white space disconnecting related things | proximity: related closer than unrelated |
| 13 | Redundant labels ("Price") | delete; the value explains itself |
| 14 | Misleading title; price buried | exact title; price by the title and on the action |
| 15 | Aggressive buy button; quantity selector far from it | calm solid button; quantity beside it; total on the button |

## Premium rules

- **Systematize components**: each works across every content it can hold (any image, any length).
- **Cohesive visual system**: imagery, icons, radii, shadows from one set.
- **Information proximity**: what decides together sits together.
- **Reduce friction**: total price / time / count on the action; no post-click surprises.

## UX upgrades to consider

- **Sticky-header card**: content area scrolls under a pinned title + key fact.
- **Sticky bottom actions**: primary controls visible the whole journey.
- **Predefined options**: one-tap presets from real usage (500 g, 1 kg; 15 min, 1 h), free entry kept.

## Code behind the UI (seven laws)

1. **Main path visible**: guard clauses, early return, no deep nesting.
2. **Name by meaning**: `pendingOrder`, not `data`.
3. **Boundary external systems**: map outside shapes to your own types at the edge (wasm, IPC, HTTP, DOM measurements).
4. **Invalid states unrepresentable**: enums and distinct types over optional fields and flag combinations (a component state is one enum, not five booleans).
5. **Decisions apart from actions**: pure function decides (tone, layout, next state); a thin layer performs (draw, write, send). Decision testable without a GPU or DOM.
6. **Useful errors**: machine code + human sentence + context; no secrets logged.
7. **Focused changes**: one objective per change; small, reviewable, revertible.
