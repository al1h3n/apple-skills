# Gestures and touch feedback

Timings and spring maths: liquid-glass `motion.md`. Rules of motion: apple-ui `motion-principles.md`. Here: what is specific to fingers.

## 1. Standard gestures (do not repurpose)

| Gesture | Means | Notes |
|---|---|---|
| Tap | activate | respond on touch-down (highlight), act on touch-up inside |
| Long-press (≈ 0.5 s) | context menu / preview | haptic at trigger; never the only path to a command |
| Swipe row sideways | row actions | leading = positive (mark, pin), trailing = destructive; full swipe performs the first |
| Edge swipe from leading | back | never block; custom leading-edge gestures conflict |
| Swipe down on a sheet | dismiss / lower detent | follows the finger 1 : 1, rubber-bands past limits |
| Pull down in a list | refresh | spinner in the bar area; haptic at threshold |
| Pinch / double-tap | zoom | double-tap zooms to the tapped point |
| Swipe up from bottom edge | Home / switcher | keep the bottom 34 pt free of horizontal drags |
| Swipe down from top-leading / top-trailing | Notification Center / Control Center | keep top corners free of drags |
| Two-finger tap, three-finger swipe | undo / redo (system) | also provide buttons |

Every gesture → a visible control that does the same.

## 2. Direct manipulation

- **1 : 1 while the finger is down.** Thing under the finger stays under it. No smoothing lag above ≈ 0.15 s.
- **Release keeps velocity.** Sheet, page, card continue with the finger's speed, then settle (spring 0.45 s, bounce 0.1): overshoot 1–2 %, no more.
- **Rubber band** past a limit: resistance grows, returns without bounce.
- **Projected end point** decides the snap (detent, page): where the flick would land, not where it was released.
- **Interruptible always.** Touch during an animation grabs it at its current place.
- **Dragged glass**: lifts ×1.08–1.10, face thins; does not stretch.
- **Reorder / resize** (edit modes): displaced neighbour waits ≈ 0.6 s under the dragged item, then moves in ≤ 80 ms. Slow to decide, fast to move.
- **Value under the finger** (slider, dial, scrubber): updates in place every step. No rolling digits, no lag.

## 3. Touch feedback

| Moment | Feedback |
|---|---|
| Touch-down on glass control | grows ≈ +5 pt (cap 15 %), lights from the finger; peak ≈ 50 ms |
| Touch-down on a row / fill button | highlight fill at once |
| Touch-up | action fires; light fades τ ≈ 90 ms |
| Finger slides off | cancels; control returns |
| Switch / slider knob | becomes a clear lens while held |
| Disabled control | no response at all |

Never delay the highlight to "wait and see" if it is a scroll: highlight after ≈ 50 ms hold at most, cancel on movement.

## 4. Haptics

| Haptic | Use |
|---|---|
| Selection (light tick) | value steps: picker, slider detent, segmented change |
| Impact light / medium / rigid | snap to detent, drop after drag, toggle |
| Success / warning / error | result of a task; pair with a visible state |

Sparing. Never on every tap. Never as the only signal. Respect the system haptics setting.

## 5. Reach

- Bottom third = primary. Top corners = rare. One-handed use assumed on iPhone.
- 44 × 44 minimum; 8 between targets; destructive away from the thumb's resting arc.
- iPad: both hands at the lower corners; pointer and keyboard as first-class input.

## 6. Accessibility

- VoiceOver: every control labelled, traits set, logical order; custom gestures get actions (`accessibilityAction`).
- Switch Control / Full Keyboard Access: everything focusable, visible focus ring.
- Reduce Motion: no travel or zoom; cross-fades. Prefer Cross-Fade Transitions honoured.
- No time-limited gestures; long-press duration follows the system setting.
- Drag operations have a non-drag alternative (buttons, menu).
