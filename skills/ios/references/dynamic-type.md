# Dynamic Type

Default ramp (size "Large"), weights, macOS ramp, licences: apple-ui `typography.md`. This file: how the ramp scales and how layouts survive it.

## 1. Sizes by setting [Apple HIG]

Points, size / leading.

| Style | xSmall | Small | Medium | **Large** | xLarge | xxLarge | xxxLarge |
|---|---|---|---|---|---|---|---|
| Large Title | 31/38 | 32/39 | 33/40 | **34/41** | 36/43 | 38/46 | 40/48 |
| Title 1 | 25/31 | 26/32 | 27/33 | **28/34** | 30/37 | 32/39 | 34/41 |
| Title 2 | 19/24 | 20/25 | 21/26 | **22/28** | 24/30 | 26/32 | 28/34 |
| Title 3 | 17/22 | 18/23 | 19/24 | **20/25** | 22/28 | 24/30 | 26/32 |
| Headline | 14/19 | 15/20 | 16/21 | **17/22** | 19/24 | 21/26 | 23/29 |
| Body | 14/19 | 15/20 | 16/21 | **17/22** | 19/24 | 21/26 | 23/29 |
| Callout | 13/18 | 14/19 | 15/20 | **16/21** | 18/23 | 20/25 | 22/28 |
| Subheadline | 12/16 | 13/18 | 14/19 | **15/20** | 17/22 | 19/24 | 21/28 |
| Footnote | 12/16 | 12/16 | 12/16 | **13/18** | 15/20 | 17/22 | 19/24 |
| Caption 1 | 11/13 | 11/13 | 11/13 | **12/16** | 14/19 | 16/21 | 18/23 |
| Caption 2 | 11/13 | 11/13 | 11/13 | **11/13** | 13/18 | 15/20 | 17/22 |

Accessibility sizes (Larger Text on), Body: AX1 28/34, AX2 33/40, AX3 40/48, AX4 47/56, AX5 53/62. Large Title tops out at 60/70.

Minimum anywhere: 11 pt.

## 2. Layout rules

- Text containers grow in height. No fixed heights on anything holding text.
- Rows: min height 44–52, grow with content. At AX sizes: label above value (vertical stack), not side by side.
- Bars do **not** scale their glyphs with Dynamic Type: tab bar and toolbar items stay; long-press shows the Large Content Viewer (HUD with the item enlarged). Provide `largeContentTitle` / image.
- Buttons: label wraps to two lines before truncating. Width grows; keep 16 padding.
- Icons beside text scale with it (SF Symbols with a text style; `@ScaledMetric` for custom sizes and spacing).
- Truncation last resort: tail ellipsis, never mid-word clipping; full text reachable.
- Multi-column grids drop to fewer columns as size grows.
- Scroll views everywhere content can outgrow the screen.

## 3. Implementation

```swift
Text("Title").font(.title2)                       // text style, never a fixed size
@ScaledMetric(relativeTo: .body) var icon = 22.0  // custom metric that follows Body
ViewThatFits { HStack { label; value } ; VStack(alignment: .leading) { label; value } }
@Environment(\.dynamicTypeSize) var size          // size.isAccessibilitySize → switch layout
```

UIKit: `UIFont.preferredFont(forTextStyle:)` + `adjustsFontForContentSizeCategory = true`; `UIFontMetrics(forTextStyle:).scaledValue(for:)`.

Web apps for iPhone: `font: -apple-system-body` (and `-apple-system-headline`, …) follows the setting; or size in `rem` and respect the user's root size. Never set `html { font-size: 62.5% }`-style resets that defeat it; never `user-scalable=no`.

## 4. Test

- Smallest (xSmall), default, largest standard (xxxLarge), AX3, AX5.
- With Bold Text on (weights step up), Increase Contrast, Button Shapes.
- Longest locale (German, Russian) at AX3.
- Nothing clipped, overlapped or unreachable; primary action still on screen or one scroll away.
