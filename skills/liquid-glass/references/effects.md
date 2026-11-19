# Effects: system-level moments, measured

Bigger than one control: backdrop, banners, assistant, pages. All measured frame by frame on iOS 27 recordings; τ = exponential time constant. Use as recipes; springs in `motion.md` §1–2.

## 1. Backdrop depth (app closes → Home returns) [M1]

Three layers, three speeds. Scale about screen centre.

| Layer | Motion | Timing |
|---|---|---|
| App card | shrinks into its icon | gone 0.12 s |
| Foreground (icons, dock) | scale 0.87 → 1 | launched, **τ 110 ms**, done 0.6 s |
| Foreground blur | 18 pt → 0 | launched, **τ 60 ms** (2× faster than scale → readable while moving) |
| The app's own icon | flies from card into its slot, **sharp** among blurred neighbours | lands ≈ 0.3–0.37 s |
| Wallpaper | scale 1.10 → 1 | spring 1.0 s, bounce 0, from rest; last thing to stop |
| Wallpaper blur | 8 pt held 0.3 s, then → 0 | τ 140 ms, gone 0.7 s |
| Wallpaper gain | ×0.8 → ×1.09 while blurred → 1 | back by 0.95 s |

Foreground comes **toward** viewer, wallpaper falls **away** → camera pulling back. Reads as stepping out, not a card vanishing.

Open (app launches): foreground → 0.89, blur → 18 pt, gain ×0.8, all within 0.1 s; then covered. Leaving 6× faster than returning.

```
Backdrop { scale, blur_pt, gain }      applied where content is copied in
return : scale 1.10 → 1   spring(1.0, 0) from rest
         blur  8 pt → 0   hold 0.3 s, then τ 140 ms
         gain  0.8 → 1    τ ≈ 100 ms
fore   : scale 0.85 → 1   launched τ 110 ms   (spring 0.69, 0)
         blur 18 pt → 0   launched τ 60 ms
leave  : scale → 0.89, blur → 18 pt, gain → 0.8 in 0.1 s
```

Reduce Motion: no scale; blur cross-fade only. Apple removed the wallpaper zoom in 26 and restored it in 27.3 [Apple]: depth cue outranks simplification.

## 2. Notification banner [M4][M7]

**Origin = where notifications live on that screen.** Home Screen: under the leading status-bar item (clock). Lock Screen: bottom centre, between the two round buttons. Never slid from off-screen. Never faded.

| Phase | What | Timing |
|---|---|---|
| Anticipation | strip 43 × 11 pt (11 % of final width) under the clock; drops 33 pt at constant size | 0.1 s |
| Grow | frame → final. **Width leads height** (wide and flat first). Curved path, small dip below rest | spring 0.42 s, bounce 0.2, from rest; 1.2 % overshoot, settle 0.4 s |
| Material | clear + strongly lensing while small (icons under it magnified); frost and tone build with size | with size |
| Content | inside from frame 1, scaled with the shape. Readable at 0.65 scale. Never clipped, never popped in | none |
| Rest | large Regular → **follows colour scheme**, not backdrop. r 24–26. Heavy frost | none |
| Leave | back to the same anchor. Height collapses faster than width; ends as a line 51 × 4 pt that rises and fades in the last 4 frames | launched τ 75 ms; 95 % at 0.2 s |

Lock Screen variant: scale 0.25 → 1 as a spring from rest, bounce 0, ≈ 0.23 s; then drifts up 6 pt and settles (0.15 s); clear dark glass.

## 3. Assistant island (Siri) [M6][M7]

Three shapes, three meanings. Shape = status; no label needed.

| State | Shape (pt) | Motion |
|---|---|---|
| Summon | island 126 × 37 → **drop** ≈ 165 × 130, round | launched τ 60 ms; 95 % at 0.18 s; no overshoot |
| Drop clears | lower ≈ 40 % turns to clear glass, bottom up | 0.2 s. Clear part = strong lens: widgets under it appear inside, **minified** |
| Listening | drop breathes ±1 % | light line on the black/clear boundary: white core, blue left end, orange-red right end; width and tilt follow the voice |
| Working | drop → **pill** ≈ 246 × 61 | launched τ 75 ms; 95 % at 0.25 s. Line collapses to a dot first. Clear part goes black |
| Working ring | Ø ≈ 22 pt at leading end | dot → ring → 7 glowing spheres in 0.3 s. **3D ring**: tilts and precesses; front spheres larger and brighter; pass behind each other. Never a flat spinner |
| Speaking | pill → **bar**, screen width − 16, 79 tall | spring 0.5 s, bounce 0.15, from rest; ≈ 2 % overshoot. Status-bar items blur and fade under it in 0.15 s (not pushed) |
| Status text | grey ≈ 55–60 % white, regular | swap: `motion.md` §3 text swap |
| Answer | white, one size up, starts at ring's left edge | ring + status blur out 100 ms; 2 dark steps; answer sharpens 130 ms |
| Rim light | soft band on the black/clear boundary, right of centre | starts 0.65 s after the answer; **blue-white ↔ orange, period 1.6–1.9 s**, repeats; glow rises 10 pt |

Material: black at top → clear glass over the lower third. Thin light rim only on the clear part.

## 4. Answer card [M7]

One glass shape, screen width minus margins, r ≈ 40.

- **Face: opaque black behind text, fading to clear over the lower third.** Legibility by material, no scrim on the wallpaper.
- **Bottom lip: clear lens 25–30 pt deep.** Content under it bent and compressed. Thin bright rim on bottom edge and corners.
- Text: white, regular, ≈ Title 3, left aligned.
- Embedded card: **not glass**. Light fill (white 12–15 %), large continuous corners, nearly full width. Or the app's own opaque card.

| Step | From first growth frame |
|---|---|
| island starts growing; status icons blur out | 0 |
| line in the embed's colour lights the bottom rim (anticipation by light) | 0.13 s |
| line thickens to a strip | 0.17 s |
| shape at full size | 0.23 s |
| text line 1 sharp; each next line +50 ms, top first | 0.23–0.33 s |
| embed slides up 4 pt + sharpens | 0.33–0.43 s |
| numbers in the embed roll into place | 0.3–0.43 s |

Collapse: content scales and blurs **with** the shape; back to island in 0.17–0.2 s.

Reveal order = reading order: shape → colour hint → words → rich card → numbers.

## 5. Surface from its button [M7][M8]

`motion.md` §3 has the timing. Frame-level detail:

1. Press: glass circle lights (to near-white on this small button), grows ≈ 10 %, glyph fades.
2. Small egg-shaped blob, no glyph, already moving toward where the menu sits.
3. Dome: fully round top, no corners. Rows already inside, soft, **warped at the growing edge like under a lens**.
4. Corners tighten to the menu's own (≈ 30–34 continuous). Rows sharp.

Button is gone while the menu is open. Neighbours stay. Close: reverse on recede; last remnant sits at the button; glyph sharp one frame later.

Share sheet: same. First content appears directly above the button, small, rises; header first, rows follow one after another (own springs). 0.27 s open, 0.2 s close.

## 6. Control Center [M8]

- **Open**: Home blurs + dims in ≈ 65 ms. All controls appear together, soft; sharp at 0.2 s; settled 0.45 s.
- **Pages** (controls ↔ Now Playing ↔ Connectivity), finger flick: outgoing leaves in 1–2 frames, streaked. Incoming: launched with the finger's velocity, τ 60–67 ms, landed 0.2 s, 1–2 % overshoot, rows lag ≈ 1 frame each.
- **Dismiss**: tiles stay in place and dematerialize (face first, rim last), 0.18 s. Home sharp at 0.28 s.
- **Edit**: "−" badge top-left, resize grip bottom-right. Resize follows the finger directly (circle → capsule). Displaced neighbour waits 0.6 s (two shapes overlap, lower visible through upper), then moves in ≤ 80 ms. Dragged control: lift 8–10 %, more translucent, tinted by what is under it. No stretch.
- Tiles: Clear glass over the blurred, dimmed Home. State colour only in small icon circles (orange, blue) or white circle + dark glyph. Inactive = grey glyph, no fill.

## 7. Camera and Photos [M8]

| Moment | What |
|---|---|
| Mode strip | names on one dark glass capsule; selection = lighter fill pill staying centred; strip follows finger; selected name turns accent **0.2 s after** settle |
| Shutter → record | colour interpolates white → pink → red (0.17 s). Disc → rounded square in a glass ring: one shape, 0.12 s, −35 % size |
| Side buttons | materialize in place: soft dot at 0.5 → full in 0.13 s, solid 0.25 s. Do not bud off the shutter |
| Leaving items | fade out in place, 0.2 s. Nothing slides |
| Zoom dial | chips → arc in 0.15 s: ticks, then labels, on a dark band dimming the picture. Turns 1:1 with finger. Value at top centre in accent, updates in place |
| Viewer toolbar → edit tools | cross-fade in place 0.13 s; top bar 0.1 s later; secondary items in 0.1 s steps; whole change 0.55 s, first-needed tools sharp at 0.15 s |
| Tinted action ("Clean Up") | yellow capsule appears only after there is something to act on; ≤ 80 ms, rises 8 pt; dark label; lit top rim, darker lower edge. After tap: replaced by a progress line + round cancel; accent moves to the progress line |

## 8. Widgets

- Home Screen widget: r 28, thin inner rims [Kit]. Clear style: tile is clear with a refraction band ≈ 14 pt on a large widget, 3–4 pt on a 62 pt icon; rim takes the backdrop's colour [M2].
- Page swipe: widgets and icons keep their shape; the lens re-renders the wallpaper every frame → motion visible inside each tile [M2].
- Lock Screen widgets: not glass. Blur 40 + additive white [Kit].
- Animated content (weather, live values): lives in the content slot above the glass; the glass does not animate with it. Numbers that change while watched roll once on arrival [M7], update in place under a finger [M8].

## 9. Checklist for any new effect

- Has a source and returns to it.
- Launched if a finger sent it; from rest with ≤ 0.2 bounce only if it brings something to read.
- Leaves faster than it arrives, no bounce.
- Content inside from frame 1; soft while moving; sharp on landing.
- One thing stays sharp and coloured: the one that matters.
- Reduce Motion variant written down.
