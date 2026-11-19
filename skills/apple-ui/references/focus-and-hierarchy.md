# Focus and hierarchy

HIG themes: **clarity** (legible, precise), **deference** (UI never competes with content), **depth** (layers + motion convey hierarchy).

## 1. Decide the job

Write: "This screen lets <person> <verb> <object>." Needs "and" → two screens, or primary + secondary.

| Screen | Job | Primary action |
|---|---|---|
| Search results | pick a place | Directions on best match |
| Place card | decide to go | Directions (solid) |
| Settings | change behaviour | none: toggles act at once |
| AI quick card | get one answer | the answer's own action |
| Landing hero | understand the promise | one CTA |
| Product page | decide to buy | Add, with total price on the button |

## 2. Attention budget

Eye sees one thing first, then scans. Spend in order:

1. Hero content (map, photo, product, answer).
2. Primary action: solid colour, one per screen, **appears when usable**.
3. Navigation + context (title, back, close).
4. Secondary actions: fills, no colour.
5. Everything else: folded, "More…", Settings, credits.

Squint test: blur the screenshot → hero and primary action still stand out.

## 3. Progressive disclosure

- > ~6 options → common ones, then "More…".
- Explanation: one line under the control. Rest behind "Learn more".
- Expert features one level deeper than the core job. Never on a demo's first screen.
- Required attribution visible where the licence demands; full list folded.

## 4. Proximity (information that decides together sits together)

- 8 pt inside a group, 16–24 between groups. Too much white space disconnects related things as surely as too little crowds them.
- **Trust signals next to the object they vouch for**: rating + review count right under the title, not three blocks down.
- **Price where the decision happens**: near the title and on the action. Not buried.
- **Controls for one decision in one place**: quantity selector beside the buy button, not floating elsewhere.
- Title names the thing exactly (no size or count that the selector then contradicts).
- Label only what needs one. "Price", "Title", "Quantity" above an obvious value = noise.

## 5. Grouping without boxes

- Inset grouped list: one fill per group, hairlines between rows inset to text.
- No card round every row, pill round every value, circle round every icon. Removing a shape loses no meaning → remove it.
- Dividers light (separator colour, hairline). Dark lines chop the page into blocks.
- One container per stack: a glass panel holds plain rows, never more glass.

## 6. Systems, not instances

- Design each component for **every content it will hold**: longest name, biggest number, darkest and lightest image, no image, RTL.
- Icons over photos need their own contrast (material or scrim behind them); do not rely on the photo.
- One icon family: one weight, one style (all filled or all outline), one colour logic.
- One imagery style across a catalogue: same background, lighting, crop, scale.
- Align to few edges. Same margins everywhere; a shifted grid reads as unstable.

## 7. Colour as signal

- Accent on the one primary action. Semantic colour on glyphs where colour carries meaning.
- Same category → same colour, everywhere.
- Never colour two competing actions.
- Saturation serves the product: a loud palette round a product photo beats the photo.

## 8. Copy is hierarchy

Shorter = higher. 5-word title beats 15. Benefit first, mechanism second.

## 9. Keep context while moving

- Long content: sticky header (title, key fact) on a scrolling card.
- Action stays reachable: sticky bottom bar with the primary action for the whole journey.
- Common choices as one-tap presets (sizes, amounts, times) taken from real usage; free entry stays available.

## 10. Empty, loading, error

- Loading: skeletons shaped like the final layout. Own data first, third-party embeds later. Never a full-screen spinner.
- Empty: what will appear + one action.
- Error: what happened in plain words + what to do. Keep the last good data on screen.

## 11. Laws worth knowing

| Law | Consequence |
|---|---|
| Hick | fewer visible choices → faster decision |
| Fitts | primary action large and near (bottom on phones, by the content on desktop) |
| F / Z scanning | meaning in the first words of titles and rows |
| Serial position | best match first, CTA last |
| Aesthetic-usability | polished reads as easier; polish is not optional |
