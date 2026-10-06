# Typography

## 1. App UI: Apple HIG Dynamic Type (default "Large" size)

| Style | iOS / iPadOS size / leading | Weight | macOS size / leading |
|---|---|---|---|
| Large Title | 34 / 41 | Regular (Bold in nav bars) | 26 / 32 |
| Title 1 | 28 / 34 | Regular | 22 / 26 |
| Title 2 | 22 / 28 | Regular | 17 / 22 |
| Title 3 | 20 / 25 | Regular | 15 / 20 |
| Headline | 17 / 22 | Semibold | 13 / 16 (Bold) |
| Body | 17 / 22 | Regular | 13 / 16 |
| Callout | 16 / 21 | Regular | 12 / 15 |
| Subheadline | 15 / 20 | Regular | 11 / 14 |
| Footnote | 13 / 18 | Regular | 10 / 13 |
| Caption 1 | 12 / 16 | Regular | 10 / 13 |
| Caption 2 | 11 / 13 | Regular | 10 / 13 |

Rules:
- Use these sizes only. Emphasis = a heavier weight of the same style (Semibold/Bold), not a
  new size.
- Two weights per screen is usually enough (Regular + Semibold), a third (Bold) for titles.
- Secondary text = same size or one step down **and** secondary colour (not both much
  smaller and grey).
- Section captions: Caption 1 Semibold, uppercase, +2 % tracking, secondary colour.
- Numbers that change (times, prices, distances): tabular figures.
- Support Dynamic Type: layouts must reflow at larger sizes (no fixed-height text boxes).
- Web desktop apps may use iOS sizes (17 body) for touch-first maps, or macOS sizes for dense
  pro tools; choose one ramp per product.

## 2. Marketing pages: a modular scale

Pick a base (16–18 px) and a ratio, then use only its steps.

| Ratio | Feel | Steps from 17 |
|---|---|---|
| 1.25 (major third) | calm, docs | 11, 14, 17, 21, 27, 33, 42, 52, 65 |
| 1.333 (perfect fourth) | editorial, product | 10, 13, 17, 23, 30, 40, 54, 72, 96 |
| 1.5 (perfect fifth) | dramatic hero | 11, 17, 26, 38, 57, 86, 129 |

- Display serif at 1.0–1.05 line height, −1 % tracking; body sans at 1.4–1.5.
- Line length 45–75 characters; hero sub-heads ≤ 2 lines on desktop.
- One italic word in a serif headline gives an editorial accent ("See your city *before* …").

## 3. Fonts and licences

- **SF Pro, SF Compact, SF Mono, New York**: licensed for designing and mocking up UIs for
  Apple platforms. Do not embed them in web fonts, Android or Windows builds. On the web use
  `font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", sans-serif;`
  — Apple devices draw SF from the OS, others get their native UI font.
- Free to ship (SIL OFL): Inter, Inter Display, Geist, Manrope, IBM Plex Sans; serif:
  Instrument Serif, Newsreader, Source Serif 4, Fraunces, Lora; mono: JetBrains Mono, Geist Mono.
- Claude.ai-like pairing for free: Instrument Serif or Newsreader (headings) + Inter (UI).
- Always self-host OFL fonts (no third-party font CDN tracking), subset to the scripts you
  ship (Latin + Cyrillic for Kazakhstan/Russia), `font-display: swap`.

## 4. Web tokens (example)

```css
:root {
  --font-ui: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --font-display: "Instrument Serif", "Newsreader", Georgia, serif;
  --text-caption2: 11px/13px var(--font-ui);
  --text-footnote: 13px/18px var(--font-ui);
  --text-subheadline: 15px/20px var(--font-ui);
  --text-body: 17px/22px var(--font-ui);
  --text-title2: 600 22px/28px var(--font-ui);
  --text-large-title: 700 34px/41px var(--font-ui);
}
```
