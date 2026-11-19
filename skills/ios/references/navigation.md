# Navigation and structure

## 1. Pick the structure

| App shape | iPhone | iPad |
|---|---|---|
| 2–5 peer sections | tab bar (bottom, floating) | tab bar (top capsule) ↔ sidebar (`sidebarAdaptable`) |
| Deep hierarchy | navigation stack per tab | split view: sidebar · list · detail |
| One canvas + tools (map, camera, editor) | content full-bleed, floating glass clusters, sheets for detail | same + inspector |
| Single flow | stack, no tab bar | stack in a centred column |

> 5 sections → the fifth is "More" or the structure is wrong. Tabs are places, never actions.

## 2. Tab bar [Kit]

- iPhone: floating capsule, bottom; 2–5 tabs; symbol + label (Semibold 10). Selected = burn capsule + accent.
- **Search role**: separate round button at the trailing end; tap → field takes the bar.
- **Prominent tab**: one tinted tab apart from the rest (a cart, compose).
- Minimise on scroll down; accessory (mini player) sits above or merges inline.
- iPad: 491 × 44 capsule at the top centre, leading sidebar toggle. Sidebar 320 wide.
- Foldable (iPhone Duo, landscape / inner display): vertical bar, 48 wide, leading edge.
- Stays visible on push. Hide only in immersive content (video, camera) and bring it back on tap.

## 3. Toolbars [Kit]

- Top: 54 tall under the status bar; leading = back / close; centre = title (Semibold 17) or 2-line title; trailing = ≤ 3 items, the primary one tinted.
- Large title: 34 Bold, collapses into the bar on scroll.
- Bottom (iPhone): 84 tall; 48 × 48 buttons; variants with page dots or a search field.
- Group related items in one capsule; split unrelated groups with space.
- **Mode change** (select, edit): buttons morph, **title slot stays**.
- No opaque bar backgrounds. Content scrolls under; scroll edge effect soft, or hard under pinned headers.

## 4. Search placement

| Context | Place |
|---|---|
| Search is a section | tab with search role |
| Search within a screen, iPhone | bottom toolbar field (thumb reach) |
| iPad / wide | top trailing field |
| Sidebar app | field at the top of the sidebar |

Focused: field expands, Cancel / close appears, suggestions in the content area. Results replace content in place.

## 5. Modality

| Present as | When | Dismiss |
|---|---|---|
| Push | going deeper in a hierarchy | back button + edge swipe |
| Sheet, medium detent | task that needs the context behind | drag down, close button |
| Sheet, large | self-contained task; turns opaque | Cancel / Done in its bar |
| Full-screen cover | immersive (camera, player, onboarding) | explicit close |
| Popover (iPad) | small choice anchored to a control | tap outside |
| Menu | 2–8 commands | tap outside, choose |
| Alert | must decide now | button only |

- Sheet: glass at medium (inset 6, r 38 top / 58 bottom), grabber 60 × 4. Dims content 20 %.
- Unsaved changes → confirm before drag-dismiss.
- One modal layer. A sheet on a sheet = redesign.
- Sheets and menus grow from the control that opened them when one exists (zoom transition from source).

## 6. Back, state, deep links

- Edge swipe back always works; never trap with a custom gesture on the leading edge.
- Restore scroll position and selection when returning.
- Every screen reachable by a link; state restoration after relaunch.

## 7. iPad specifics

- Sidebar (320) · content; optional inspector trailing. Sidebar collapses to the tab bar in compact width.
- Window controls (close, minimise, full screen) in the top-leading corner in windowed mode: keep 27 × 22 glass capsule clear; do not put a back button under it.
- Menu bar exists with a hardware keyboard: expose commands with shortcuts.
- Pointer: hover states on every control; context menus on secondary click.
- Popover instead of sheet for small tasks in regular width; adapts to a sheet in compact.

## 8. Safe areas, sizes

- Reference canvas 402 × 874 (iPhone 17 Pro / 18 Pro); Pro Max 440 wide; iPad 11" 834 × 1210; 13" 1032 × 1376.
- Top inset: status bar 62 incl. Dynamic Island (95 × 36.67). Bottom: home indicator area 34 (iPad 21).
- Bottom-anchored panels match the display corner (r 58–62 at the bottom corners).
- Layout by size class and available width, never by device name. Resizable everywhere (iPad windows, iPhone mirroring).
