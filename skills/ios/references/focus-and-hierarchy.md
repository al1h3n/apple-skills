# Focus and hierarchy

Apple's three themes (HIG): **clarity** (legible, precise, purposeful), **deference** (the UI
helps people understand content and never competes with it), **depth** (layers and motion
convey hierarchy). Everything below is how to apply them.

## 1. Decide the job

Write: "This screen lets <person> <verb> <object>." If you need "and", you have two screens
or a primary + secondary split. Examples:

| Screen | Job | Primary action |
|---|---|---|
| Search results | pick a place | Directions on the best match |
| Place card | decide to go | Directions (solid) |
| Settings | change how the app looks/behaves | none (toggles act immediately) |
| AI quick card | get one answer | the answer's own action pill |
| Landing hero | understand the promise | "Open the map" |

## 2. The attention budget

People see one thing first (the eye's sharp focus covers a tiny area), then scan.
Spend the budget in this order:

1. **Hero content** (map, photo, the answer text).
2. **The primary action** (solid colour, one per screen).
3. **Navigation and context** (title, back, close).
4. **Secondary actions** (fills, no colour).
5. **Everything else** (folded, in "More…", in Settings, in credits).

Squint test: blur the screenshot; the hero and the primary action must still stand out.

## 3. Progressive disclosure

- Lists longer than ~6 options: show the common ones, then "More…" (chips, menus).
- Explanations: one line under the control (footnote). The rest goes behind "Learn more".
- Expert features (layers, monitoring, time travel) live one level deeper than the core
  job; never on the first screen of a demo.
- Legal/credits text: required attribution visible where the licence demands (e.g. "©
  OpenStreetMap" on a map), the full list folded in "Sources & credits".

## 4. Grouping without boxes

- Proximity first: 8 pt inside a group, 16–24 pt between groups.
- Inset grouped list = one fill for the group, hairlines between rows, inset to the text.
- Do not put a card around every row, a pill around every value, a circle around every
  icon. If removing a shape loses no meaning, remove it.
- One container per stack: a glass panel holds plain rows; it never holds more glass.

## 5. Colour as signal

- Accent on the **one** primary action per screen; semantic colour on glyphs where colour
  carries meaning (yellow = people, blue = you, green = clean air / open now, red = destructive
  or fastest-but-dirty).
- Category colour must be consistent: same category, same colour, everywhere.
- Never colour two competing actions.

## 6. Copy is hierarchy

Shorter text is higher in the hierarchy. A 5-word title beats a 15-word one. Lead with the
benefit ("Walk the street with less smoke"), not the mechanism ("PM2.5-weighted Dijkstra").

## 7. Empty, loading, error

- Loading content: skeleton pills shaped like the final layout; our own data appears first,
  third-party embeds fill in later.
- Empty: say what will appear and give one action ("No saved places yet · Save one from any
  place card").
- Error: what happened, in plain words, and what to do ("Can't reach the air sensors. Showing
  the last reading from 14:20.").

## 8. Research worth knowing

- **Hick's law:** decision time grows with the number of choices → fewer visible options.
- **Fitts's law:** big, near targets are faster → primary actions large and in reach
  (bottom on phones, near the content on desktop).
- **F/Z scanning (NN/g):** people scan headings and the left edge → put meaning in the first
  words of titles and rows.
- **Serial position:** first and last items are remembered best → best match first, CTA last.
- **Aesthetic-usability effect:** polished UIs are perceived as easier → polish is not optional.
