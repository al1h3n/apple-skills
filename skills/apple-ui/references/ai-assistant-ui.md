# AI assistant UI

Measured reference for shapes and timings: liquid-glass `effects.md` §3 (assistant island), §4 (answer card).

## 1. Two sizes, one assistant

| Size | When | Shape |
|---|---|---|
| **Quick card** | one question, one answer | one glass shape: **opaque black behind the text, fading to clear glass over the lower third**, clear lens lip at the bottom. Phone: grows from the Dynamic Island. Desktop: centred or from the assistant button. Large answer text, one embedded result card, 1–2 actions |
| **Full window** | planning, several turns | docked panel beside content (desktop right; phone large sheet). Content stays visible so results land on it |

Expand on the card → window. Collapse in the window header → card. Same session.

## 2. Entry points (same session)

1. Assistant button on the main screen → quick card.
2. Typing in search with the AI switch on → inline "Ask <Assistant>"; Enter sends to the card.
3. "Ask <Assistant>" on any object card → card with that object attached.
4. Keyboard shortcut (⌘/Ctrl + J); press-and-hold on the button → voice.
5. Proactive tips → full window on the relevant session.

## 3. States = shapes

Shape is the status indicator. No label needed for the first two.

| State | Shape | Cue |
|---|---|---|
| Listening | round drop | light line across it reacting to the voice; lower part clear, showing what is under it |
| Working | compact pill | ring of 7 glowing spheres on a tilting 3D ring + one status word |
| Answering | wide bar / card | text; then a slow light breathing on the lower rim |

Changes between shapes ≤ 0.25 s, launched. Only the last (carrying text) has a little bounce.

## 4. Window anatomy

- **Header**: glyph · session switcher · New chat · Collapse · Close.
- **Thread**: person's messages right-aligned in fills. Assistant: plain text (no bubble) → cards (timeline, place, route, table) → actions (one solid primary) → 2–3 follow-up chips.
- **Progress line**: "Checked 46 places, weather and air quality" + working ring. Never a spinner. Status words swap by blur (65 ms out, 120 ms in), never slide.
- **Answer lands**: ring gives way to text; lines sharpen top first, 50 ms apart; embedded card slides up 4 pt last; its numbers roll once. A line in the card's colour lights the rim ≈ 0.1 s before the card appears.
- **Composer**: attach · field · voice · send (solid accent circle, enabled only with text).
- **Voice**: waveform replaces the field; "Listening…"; red stop; live transcript.
- **Sessions**: Today / This week / Earlier; search; privacy footnote.
- **Money, bookings**: always a confirm step; setting "Ask before paying".

## 5. Visual rules

- AI accent (gradient light) only where the AI is involved. Colour sits on **values and glyphs** (blue + check = good, pink + flag = caution), never on row backgrounds.
- Embedded cards are **not glass**: light fill (white 12–15 %) or the app's own opaque card.
- Legibility by material (black where words are), not by a scrim over the whole screen. Dim content behind a quick card ≤ 30 %.
- Phone: status items blur away under the card; card top pure black so the island melts into it.
- Answer text Title 1 / Title 3 by space; cards Headline + Footnote.
- Collapse: content scales and blurs with the shape; 0.17–0.2 s.
