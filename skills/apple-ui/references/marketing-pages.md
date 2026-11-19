# Marketing pages people want to touch

First thing anyone touches = the marketing. Make the product look its best, honestly. Mindset: the visitor gets it **before thinking**.

## 1. Skeleton

1. **Nav capsule** over the hero: logo, 3–4 links, one CTA pill.
2. **Hero**: eyebrow pill, promise headline, one-sentence sub-head, primary CTA + ghost secondary, platform line. Real product behind it, live if possible.
3. **Touchable demo** right under the hero, labelled "Interactive demo · sample data", Reset button. Plays by itself; any input takes over. Core job only.
4. **Proof strip**: sources, partners, numbers, in a quiet tone.
5. **Feature grid** (bento): numbered, hairline borders, title + one sentence + a micro UI fragment. Real UI beats icons.
6. **Statement + product shot**: one big line, the real screen, deep shadow.
7. **Mobile section**: real phone screens at 1 : 1 (true status bar, home indicator).
8. **Closing CTA**: short line + the same primary button.
9. **Footer**: columns, real owner name, required credits.

## 2. How the eye works

- **Background + scene + hero**: one main subject with context round it.
- **Movement first**: the eye locks onto motion. A demo moving by itself wins attention before any reading. Stills should imply motion (a route mid-draw, a press mid-glow).
- **Emotion**: pair the rational claim (a number) with a feeling (a calm scene).
- **First ten seconds**: the visitor must *feel* the product's signature moments without instructions. For a glass library: press a button, open a menu out of it, scroll content under a bar and watch it stay readable.

## 3. Landing for a component library or design system

- Landing = **few signature moments, each large and touchable**. 5–6 at most. One job: "feel it".
- Everything else → a separate **gallery page**: every component, every state (rest, hover, pressed, focus, disabled, loading, empty, error), each instance movable or pinned, with material / theme / accessibility switches.
- Landing links to the gallery; gallery links to code. Do not mix: a wall of components on the landing hides the signature.
- Each moment: one line of copy, the live thing, optional "compare" toggle (old vs new, on vs off).

## 4. Interactive demo recipe

- States Idle → Action → Result; timeouts 2.5 s / 2.2 s / 5 s; loop; pointer or key jumps ahead.
- Real components, mocked data. Pause off-screen. Reduce Motion → static end state.
- Sample content famous and pretty; label it "sample".

## 5. Copy and conversion

- One primary CTA repeated (nav, hero, end), same words.
- Above the fold: what it is, who for, the one action. No feature lists.
- Every claim has visual proof on the same screen.
- Performance is design: hero image ≤ 300 KB; demo lazy-loaded after first paint; heavy engines (wasm, WebGPU) load after the static first frame, with a designed fallback.
- Measure: UTM on outbound links; one variable per test.

## 6. Mobile landing

- Rebuild, do not shrink: stacked hero, full-width buttons (56 pt), demo sized for a phone, one-column feature cards.
- Sticky bottom CTA once the hero button scrolls away.
- Check 320 / 390 / 900 / 1280 px, both themes.
