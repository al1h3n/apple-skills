# Typography

## 1. App UI ramps [Apple HIG]; default "Large" Dynamic Type

| Style | iOS / iPadOS size/leading | Weight | macOS size/leading |
|---|---|---|---|
| Large Title | 34 / 41 | Regular (Bold in nav bars) | 26 / 32 |
| Title 1 | 28 / 34 | Regular | 22 / 26 |
| Title 2 | 22 / 28 | Regular | 17 / 22 |
| Title 3 | 20 / 25 | Regular | 15 / 20 |
| Headline | 17 / 22 | Semibold | 13 / 16 Bold |
| Body | 17 / 22 | Regular | 13 / 16 |
| Callout | 16 / 21 | Regular | 12 / 15 |
| Subheadline | 15 / 20 | Regular | 11 / 14 |
| Footnote | 13 / 18 | Regular | 10 / 13 |
| Caption 1 | 12 / 16 | Regular | 10 / 13 |
| Caption 2 | 11 / 13 | Regular | 10 / 13 |

Tracking [Kit]: Body −0.43, Subheadline −0.23, Footnote −0.08, Caption 0.

What Apple's kits actually set on controls [Kit]:

| Element | Type |
|---|---|
| iOS glass button label, toolbar symbols | SF Pro Medium 17 |
| iOS toolbar title | Semibold 17; two-line: Semibold 15 + Medium 12 secondary |
| iOS tab label | Semibold 10/12; tab symbol Semibold 18 |
| iOS segmented control | 13.33/18, Semibold selected, Medium others |
| iOS menu item | Regular 17/20; control cell label Medium 12 |
| iOS notification | title Semibold 15/17, body Regular 15/18 |
| macOS control label | SF Pro Medium 13 |
| macOS window title | Bold 13/15; subtitle Medium 11 |
| macOS sidebar row | Medium 11/16; section header Bold 11 secondary |
| macOS alert | title Bold 13/16, message Regular 11/14 |

Text on glass controls is **Medium or heavier**. Regular weight only on large panels and content.

## 2. Rules

- Ramp sizes only. Emphasis = heavier weight of the same style, not a new size.
- Two weights per screen (Regular + Semibold); a third (Bold) for titles. More = inconsistency.
- One family for UI. A second (serif display) only on marketing headlines.
- Title hierarchy by size **and** weight: a title that differs from body by 1 pt and no weight guides nobody.
- Secondary = same size or one step down **+** secondary colour. Not much smaller and grey at once.
- Body line height ≥ 1.29 (17/22); long reading text 1.4–1.5. Tight leading strains reading.
- Letter spacing: never negative on small caps, badges, labels under 13 pt. Uppercase captions +2 %.
- Section caption: Caption 1 Semibold, uppercase, +2 % tracking, secondary colour.
- Changing numbers (time, price, distance): tabular figures.
- Dynamic Type: layouts reflow at larger sizes. No fixed-height text boxes.
- Web desktop: iOS sizes for touch-first products, macOS sizes for dense pro tools. One ramp per product.

## 3. Marketing: modular scale

Base 16–18 px, one ratio, only its steps.

| Ratio | Feel | Steps from 17 |
|---|---|---|
| 1.25 | calm, docs | 11, 14, 17, 21, 27, 33, 42, 52, 65 |
| 1.333 | editorial, product | 10, 13, 17, 23, 30, 40, 54, 72, 96 |
| 1.5 | dramatic hero | 11, 17, 26, 38, 57, 86, 129 |

- Display at 1.0–1.05 line height, −1 % tracking. Body 1.4–1.5.
- Line length 45–75 characters. Hero sub-head ≤ 2 lines on desktop.
- Hand-set line breaks per breakpoint in display headlines: no widows.

## 4. Fonts and licences

- **SF Pro, SF Compact, SF Mono, New York**: design and mock-up of UIs for Apple platforms only. No web-font embedding, no Android or Windows builds. Web: `font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", sans-serif;`
- Free to ship (SIL OFL): Inter, Inter Display, Geist, Manrope, IBM Plex Sans; serif Instrument Serif, Newsreader, Source Serif 4, Fraunces, Lora; mono JetBrains Mono, Geist Mono.
- Self-host OFL fonts (no third-party font CDN). Subset to shipped scripts. `font-display: swap`.

## 5. Web tokens

```css
:root {
  --font-ui: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --text-caption2: 11px/13px var(--font-ui);
  --text-footnote: 13px/18px var(--font-ui);
  --text-subheadline: 15px/20px var(--font-ui);
  --text-body: 17px/22px var(--font-ui);
  --text-headline: 600 17px/22px var(--font-ui);
  --text-title2: 600 22px/28px var(--font-ui);
  --text-large-title: 700 34px/41px var(--font-ui);
}
```
