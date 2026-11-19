# Principles: why Liquid Glass looks and moves the way it does

Read before designing anything new. Rules here outlive OS versions; numbers live in `anatomy.md`, `motion.md`, `versions/`.

Tags (whole skill): **[Apple]** Apple text/session. **[Kit]** Apple Figma kit, read node by node. **[M1]…[M8]** measured frame by frame on recording n (`sources.md`). **[RE]** reverse-engineered. **[D]** derived, tune by eye.

## 0. Priority order

**legibility > hierarchy > spatial continuity > calm > delight.**

Every 26 → 27 change traded something lower for something higher. None went the other way. Conflict between two rules → higher one wins.

## 1. Seven rules

| # | Rule | Evidence | Why |
|---|---|---|---|
| 1 | **Content first. Glass = functional layer only.** One glass per stack; fills + vibrancy above it | [Apple] HIG. [Kit] every control = `Fill + Shadow` under one `Glass Effect` | Glass competing with content defeats itself |
| 2 | **Two materials, opposite jobs.** Clear shows backdrop. Regular protects text | [M2] Clear: frost ≈ 5× lower in 27. [M3] Regular: more opaque, outlined. [Kit] panels carry 70 % white Lighten | Both "somewhat frosted" → neither works |
| 3 | **Lens carries shape, edge carries separation.** Bright rim, dark hairline | [M2] rim peak 220 vs 182. [M3] grey hairline on white. [Kit] burn hairline 0.5 pt | Shape made of blur + shadow vanishes on white |
| 4 | **Launched, not eased.** Sent by a finger → full speed frame 1, exponential landing, τ 60–75 ms | [M4] τ 75. [M6] τ 60, 75. [M8] τ 60–67 | Ease-in = delay between finger and response |
| 5 | **One shape, not two.** Surface opening from a control *is* that control growing | [M7] button → blob → dome → menu, 0.27 s. [M8] share sheet out of/into its button | Shows where it came from, where it went |
| 6 | **Calm and fast.** Small press growth, short transitions, no blown highlights, no rainbow on controls | [M3] ×1.34 → ×1.08–1.15; 0.40 s → 0.15–0.18 s. [M5] no dispersion on lens | Used hundreds of times a day; spectacle = friction |
| 7 | **Depth explains place.** Layers move at different speeds; the thing that matters stays sharp | [M1] three layers, τ 60 / 110 / 200 ms; returning icon sharp among blurred | "Where am I" answered without words |

Pattern: **effect is spent where it carries information** (edge, depth order, source). Removed where decoration only (sheen over art, blobs on every tap, tilt shimmer).

## 2. Rulings on conflicting evidence

| Conflict | Ruling |
|---|---|
| Clear got clearer, Regular more opaque | Both. Different jobs. Tune apart |
| Kit Clear frost 6 vs Home Screen icon tiles σ 0.8 pt [M2] | Controls follow kit. Icon tiles = separate pipeline (`icons.md`), frost ≈ 2 |
| "Glass merges like liquid" [Apple] vs no budding on press in 27 [M3] | Merge = structure (two controls become one capsule, group forms). Never a press reaction. Never between a control and the surface it opens |
| Press growth ×1.34 (26) vs ×1.08–1.15 (27) | Latest: +5 pt, cap 15 %. Light up, never to pure white; glyph stays readable |
| Banner from under clock [M4] vs from bottom centre [M7] | Origin = where that kind of thing lives on that screen. Host supplies anchor |
| Mostly no overshoot vs 1–2 % on banner, pages, Siri bar | Overshoot only on arrivals carrying something to read, or continuing a flick. Bounce ≤ 0.2. None on dismissals, controls, anything repeated |
| Rows lag by position, no timer [M8] vs text lines on 50 ms timer [M7] | Travelling things: own spring each, lag emerges. Text appearing in place: timed, 50 ms/line, top first |
| Toolbar light instantly, dark slowly; glyph contrast dips 0.15 s [M8] | Keep asymmetry (stops flicker). Fix dip: switch glyph colour when face crosses midpoint |
| Toolbar ignores Dark Mode over white [M8] vs banner follows scheme over dark wallpaper [M4] | By size/role: small glass with glyphs adapts to backdrop. Large glass with text follows scheme. Clear never flips |
| Apple's clock resize = two animations in a row [M8] | Do not copy. One interpolation per property, from value on screen, velocity kept |
| Dragged control keeps shape [M8] vs stretch along velocity (older builds) | Stretch 0. Lift ×1.08–1.10 + thinner face |
| Tinted label white on blue, near-black on yellow [M8][Kit] | Label by contrast against tint. Never fixed |
| Icon glass: slab over art (26) vs thin edges in art (27) | 27. 26 look = named preset only |
| HIG has no 27 revision; kit + betas do | Number differs → kit (newer). HIG *rule* ("no glass in content layer") → stands |

## 3. Disney's twelve → interface

| Principle | Interface meaning | Apple does | Default |
|---|---|---|---|
| Squash and stretch | Control changes shape under finger, volume plausible | knob → lens ×1.6 → knob [M5][Kit]; banner width leads height [M4] | lens lift per axis; no drag stretch |
| Anticipation | Small move before big one aims the eye | banner drops 33 pt for 0.1 s [M4]; button lights before sheet [M8]; colour line on rim before card [M7] | press glow leads morph by 1 frame |
| Staging | One thing matters: the sharp, coloured one | returning icon sharp [M1]; one tinted action, shown only when usable [M8] | one prominent tint; moving content soft until landed |
| Straight-ahead vs pose-to-pose | Simulation for touch, keyframes for brand moments | all controls = springs; Siri mark authored [M7] | springs everywhere, no fixed curves |
| Follow-through, overlap | Parts land at different times | wallpaper settles 1.0 s, icons 0.6 s [M1]; light outlasts shape 90 ms [M3] | spring per property and per row |
| Slow in, slow out | UI: **no slow in**, long slow out | every launched motion | `launch = 1` on tap/flick results |
| Arcs | Curved paths across screen | banner path from clock [M4] | per-axis springs with different τ |
| Secondary action | Small support, no competition | wallpaper zoom [M1]; breathing rim after answer [M6] | backdrop transform; rim sweep opt-in |
| Timing | Speed = weight, importance | close faster than open (0.2 vs 0.27 s); reflow waits 0.6 s, moves in 80 ms [M8] | recede < emerge; dwell before neighbours yield |
| Exaggeration | Once, where finger hides control | lens nearly as big as whole switch [M5] | lens only while held |
| Solid drawing | Consistent light, thickness, depth order | light top, tight shadow, bigger = thicker [Kit] | one light per scene |
| Appeal | Clear silhouette, few parts | capsules, squircles, one tint, filled glyphs | continuous corners, filled symbols |

## 4. Five rules of interface motion

| Rule | Applied |
|---|---|
| Spatial consistency | Every surface has a source, returns to it. Bar changing mode keeps title in place [M3]. Pages have order, move along it |
| Copy nature | Velocity carried over. Light from one side. Bigger glass thicker. Held things lift toward viewer |
| Speed / when not to animate | < 0.2 s for anything repeated: press 35–65 ms to peak, bar change 0.15–0.18 s, text swap 0.2 s. Value under finger updates in place, no rolling digits [M8]. Reduce Motion: no scale/travel, cross-fades stay |
| Perceived performance | Blur clears 2× faster than scale lands (τ 60 vs 110 ms [M1]) → readable while moving. Tools needed first sharpen first. Content drawn inside growing surface from frame 1 |
| Personality | From material + one or two authored moments. Never bouncing controls. Amount = setting; 26-style exaggeration = preset |

## 5. Does the person get it before thinking?

Check every screen:
- Frame after tap already shows result moving. No dead frame.
- One coloured thing = next thing to do. Appears when usable.
- Nothing read moves sideways or animates while sharp.
- Surface comes out of thing touched → no label needed.
- Readable at 0.15 s even if settle takes 0.5 s.

## 6. Next OS version

Likely to keep moving the same way → keep as **data, not code**: material numbers per size class, press growth/glow/fuse, emerge/recede/bar τ, icon edge widths + translucency, legibility thresholds + timings.

New OS ships → re-measure same eight situations → new column → copy `versions/_template.md` → update current files. Rules in §1–5 expected to hold.
