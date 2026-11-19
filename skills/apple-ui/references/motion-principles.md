# Motion principles

Product-level rules. Spring maths, measured timings, per-moment table: liquid-glass `motion.md`. System moments (backdrop, banner, assistant): liquid-glass `effects.md`.

## 1. Why things move

Motion answers "what changed, where did it come from, where did it go". Answers none → cut it.

## 2. Five rules

| Rule | Do |
|---|---|
| **Spatial consistency** | Every surface has a source and returns to it: menu out of its button, sheet out of its control, banner from where notifications live. Bars keep their title in place when changing mode. Pages have an order |
| **Copy nature** | Springs with real velocity. No instant jumps, no linear ramps. Light from one side. Held things lift |
| **Speed; know when not to animate** | Repeated hundreds of times a day (press, toggle, tab, keyboard-triggered launcher) → ≤ 0.2 s or nothing. Values under a finger update in place. Entering / leaving: fast start, soft landing. Already on screen and moving: from rest, both ends soft |
| **Perceived performance** | First frame after a tap already shows the result moving. Blur clears before scale lands. Tools needed first sharpen first. Skeletons shaped like the result buy load time |
| **Personality** | From the material + one or two authored moments. Bounce is a spice: ≤ 0.2, only on arrivals bringing something to read. Confetti-level motion annoys, and can break accessibility law |

## 3. Presets

| Preset | duration / bounce | Use |
|---|---|---|
| emerge (launched) | 0.45 / 0 → τ 72 ms | anything opening from a tap |
| recede (launched) | 0.30 / 0 → τ 48 ms | closing into its source |
| flick follow (launched with pointer velocity) | 0.45 / 0.1 | pages, sheets |
| arrival with content (from rest) | 0.42 / 0.2 | banner, answer |
| smooth (from rest) | 0.5 / 0 | tone, colour, layout change nobody sent |
| follow | 0.15 / 0.14 | drag, scrub |

Launched = start velocity `ω0 × distance` toward the target: full speed on frame 1, exponential landing. **No ease-in on anything a finger sent.**

Figma `CUSTOM_SPRING` (mass 1): stiffness = (2π / duration)², damping = 2(1 − bounce)·√stiffness. 0.5 / 0 → 157.9 / 25.1; 0.5 / 0.15 → 157.9 / 21.4; 0.45 / 0 → 195 / 27.9; 0.3 / 0 → 438.6 / 41.9.

## 4. Choreography

- **Grow from the source.** One shape: the button becomes the menu. Button absent while open. Closing lands exactly on it.
- **Press**: small growth (+5 pt, cap 15 %), lights up, glyph stays readable; over in 0.1 s. No bounce on release.
- **Content in a moving surface**: inside from frame 1, soft, sharp on landing.
- **Closing faster than opening**, no bounce.
- **Rows**: own spring each; lag emerges (≈ 1 frame per row). No timers for travelling things.
- **Text**: never slides sideways, never animates while sharp. Swap = blur out 65 ms, gap, sharpen in 120 ms. New lines 50 ms apart, top first.
- **Numbers**: roll once when they answer something the person did (route time, total). Under a finger they update in place.
- **Accent colour** lands after the motion settles, not during.
- **No source** (appears by itself) → materialize in place: scale 0.5 → 1 + sharpen, 0.13 s. No slide from nowhere.
- **One animation per change.** Never jump then animate. Interpolate once, from what is on screen, velocity kept.
- **Depth**: layers at different speeds; the thing that matters stays sharp while the rest is still blurred.
- **Loading**: skeleton shimmer ≤ one sweep per 1.5 s. Never block a whole screen.
- **AI working**: slow light, not a spinner (`ai-assistant-ui.md`).
- **Self-playing demos**: idle 2–3 s → action 2 s → result 5 s → loop; any input takes control.

## 5. Disney's twelve, short

Squash and stretch: knob → lens → knob, nowhere else. Anticipation: light before movement. Staging: one sharp, coloured thing. Straight-ahead: springs for touch; pose-to-pose: authored brand moments only. Follow-through / overlap: parts land at different times. Slow in / out: **no slow in** for finger-sent motion. Arcs: per-axis springs. Secondary action: backdrop, rim light; never competes. Timing: fast = light and unimportant; dwell before neighbours yield. Exaggeration: once, where the finger hides the control. Solid drawing: one light, consistent thickness. Appeal: clear silhouettes, few parts. Full table with evidence: liquid-glass `principles.md` §3.

## 6. Accessibility

- Reduce Motion: bounce 0, no scale or travel, no parallax or backdrop zoom, no auto-playing loops (show end state). Cross-fades stay.
- Nothing loops faster than every 3 s. Nothing flashes more than 3 times a second.
- Never animate text being read.
