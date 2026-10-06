# Motion

Full spring maths and Liquid Glass choreography live in liquid-glass-skill
(`references/motion.md`). This file is the product-level rulebook.

## 1. Why things move

Motion answers "what changed and where did it come from?". If an animation does not answer
that, cut it.

## 2. Presets (duration / bounce → Figma CUSTOM_SPRING, mass 1)

| Preset | d / b | Figma stiffness / damping | Use |
|---|---|---|---|
| smooth | 0.5 / 0 | 157.9 / 25.1 | appear, dismiss, colour, collapse |
| snappy | 0.5 / 0.15 | 157.9 / 21.4 | layout, morph, expand from a control |
| bouncy | 0.5 / 0.3 | 157.9 / 17.6 | release after press, menus popping open, AI answer arriving |
| follow | 0.15 / 0.14 | — | drag, scrub |

(stiffness = (2π/d)², damping = 2(1 − b)√stiffness)

## 3. Choreography

- **Grow from the source:** search field → results, Tourist button → chat card, card → full
  window, menu button → sidebar. Same container, matched names (Smart Animate).
- **Stagger** list rows 20–30 ms; content fades in ~80 ms after its container starts moving.
- **Press:** scale up slightly (glass grows + glows from the finger), release with bounce.
- **Loading:** skeleton shimmer at most one sweep per 1.5 s; never block a whole screen.
- **AI thinking:** a slow light (rotating gradient outline, caret glow), not a spinner.
- **Self-playing demos:** idle 2–3 s → action 2 s → result 5 s → loop; any click takes control.

## 4. Accessibility

- Reduce Motion: bounce → 0, no parallax, no auto-playing loops (show the end state), keep
  structural morphs short and flat.
- Never animate text people are reading; never loop faster than every 3 s.
