# Marketing pages that people want to touch

The first thing anyone touches is the marketing, not the product. The page must make the
product look as good as the McDonald's photo of the burger, while staying honest.

## 1. Page skeleton (proven by cursor.com, bridgemind.ai, better-auth.com, journey.com, apple.com)

1. **Glass nav capsule** over the hero: logo, 3–4 links, "Sign in", one white CTA pill.
2. **Hero:** eyebrow pill ("Live in Almaty · Paris next"), promise headline (serif display,
   one italic word), one-sentence sub-head, primary CTA (white pill) + secondary (ghost),
   platform line in small text. Full-bleed real product imagery behind, darkened toward the
   bottom.
3. **Touchable demo** directly under the hero, labelled "Interactive demo · sample data" with a
   Reset button. It plays by itself and any click takes over. Show the core job only (search
   → AI answer → route), no advanced features.
4. **Proof strip:** "Built on open data" + partner/source names in 35 % white.
5. **Feature grid (bento):** numbered 01–06, hairline borders, title + one sentence + a micro
   UI (a mini route, a chip row, a time slider). Real UI fragments beat icons.
6. **Statement + product shot:** a big serif line ("Glass that bends the city, not your
   attention.") and the real app screen with a deep shadow.
7. **Mobile section:** real phone screens at 1 : 1 (true status bar and home indicator).
8. **Closing CTA:** short serif line + the same primary button.
9. **Footer:** columns, legal line with the owner's real name and required data credits.

## 2. How the eye works (design every creative this way)

- **Background + scene + hero:** the eye needs context around one main subject.
- **Movement first:** the eye locks onto motion; a demo that moves on its own wins attention
  even before reading. Even stills should imply motion (a route mid-draw, a glow).
- **Emotion:** pair the rational claim (−38 % dose) with a feeling (an evening view, a calm
  walk).

## 3. Interactive demo recipe (Figma and code)

- Component with variants Idle → Typing → Answer; AFTER_TIMEOUT 2.5 s / 2.2 s / 5 s loop;
  ON_CLICK on the search field and the AI button jumps ahead.
- Use a sample city that is famous and pretty (Paris) even if the product launches elsewhere;
  label it "sample".
- In code: render the real components with mocked data; pause when off-screen; respect
  Reduce Motion (static end state).

## 4. Copy and conversion

- One primary CTA repeated (nav, hero, end). Same words everywhere.
- Above the fold: what it is, who it is for, the one action. No feature lists.
- Every feature claim needs a visual proof on the same screen.
- Performance is design: hero image ≤ 300 KB, demo lazy-loaded after first paint.
- Measure: every outbound link gets UTM; test one variable at a time on a 3-month horizon.

## 5. Mobile landing

- Rebuild, don't shrink: stacked hero, full-width buttons (56 pt), demo sized for a phone
  (search at the top, AI button bottom-right, hints not covering controls), feature cards in
  one column.
- Avoid widows in display headlines (set line breaks by hand per breakpoint).
