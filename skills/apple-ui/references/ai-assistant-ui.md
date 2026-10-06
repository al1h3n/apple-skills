# AI assistant UI (Siri AI style quick card + full window)

## 1. Two sizes, one assistant

| Size | When | Shape |
|---|---|---|
| **Quick card** | one question, one answer | smoked glass card (black at the top fading lighter), centred on desktop, grown from the Dynamic Island on iPhone; large answer text, one nested result card, 1–2 action pills below |
| **Full window** | planning, several turns, comparisons | docked glass panel beside the content (desktop right side, phone large sheet); content stays visible so results land on it |

Expand button on the quick card → full window; collapse in the window header → quick card.

## 2. Entry points (all open the same session)

1. The assistant button (with the AI outline) on the main screen → quick card, listening for text.
2. Typing in search with the AI switch on → inline "— Ask <Assistant>" ghost; Enter sends to the
   quick card. A suggestion row "Ask <Assistant>" does the same.
3. "Ask <Assistant>" on any object card (a place) → quick card with that object attached.
4. Keyboard shortcut (e.g. ⌘/Ctrl + J) and press-and-hold on the button for voice.
5. Notifications/proactive tips open the full window on the relevant session.

## 3. Window anatomy

- **Header:** assistant glyph · session switcher ("Evening in Almaty ⌄") · New chat (+) ·
  Collapse · Close.
- **Thread:** the person's messages in right-aligned bubbles (fill); the assistant writes plain
  text (no bubble), then **cards** (plan timeline, place, route, comparison table), then actions
  (one solid primary, the rest fills), then 2–3 follow-up chips.
- **Progress line:** "Checked 46 places, weather and air quality" with the AI glyph, instead of
  a spinner.
- **Composer:** attach (+) · field · voice (mic) · send (solid accent circle, enabled only with
  text). AI outline on the composer while the assistant is active.
- **Voice state:** waveform replaces the field, "Listening…", red stop button; transcript
  appears live in the field.
- **Sessions:** list grouped Today / This week / Earlier, search, current session selected;
  footnote on privacy ("Chats stay on our server and are never sold").
- **Money and bookings:** always a confirm step ("Confirm table"), setting "Ask before paying".

## 4. Visual rules

- The AI accent (rotating gradient outline / glow) appears only where the AI is involved.
- Dim the content behind a quick card lightly (15–30 %) — never black it out.
- On iPhone the status bar hides while the card is open (`prefersStatusBarHidden`), and the
  card's top is pure black so the island melts into it.
- Answer text Title 1 / Title 3 depending on space; cards use Headline + Footnote.
