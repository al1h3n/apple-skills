# Native Apple API (SwiftUI · UIKit · AppKit)

iOS/iPadOS/macOS/tvOS/watchOS 26+, verified against developer.apple.com docs (Oct 2026,
SDK 27). On Apple platforms **always use the system API** — it gets the real compositor
(quarter-res backdrop, SDF union, luma tracking, HDR highlights), accessibility settings,
the iOS 27 transparency slider, and future refinements for free. Never re-implement glass
with shaders inside a native Apple app.

Guard with `if #available(iOS 26, macOS 26, *)` when you still deploy to older OSes.

---

## 1. Free adoption

Rebuild with Xcode 26/27: `NavigationStack`, `NavigationSplitView`, `TabView`, toolbars,
sheets, popovers, menus, `Button`/`Toggle`/`Slider`/`Picker`/`Stepper`/`TextField` adopt the
material automatically. Remove custom backgrounds from bars and split views — they block
the system glass and the scroll edge effect.

`UIDesignRequiresCompatibility = YES` (Info.plist) keeps the pre-26 look for one more
cycle; Apple has signalled it goes away with Xcode 27-era requirements. Plan to adopt.

## 2. SwiftUI core

```swift
// Glass configuration (struct Glass)
.regular            // default, adaptive
.clear              // media overlays only
.identity           // no effect (transition endpoint / conditional)
.tint(_ color: Color?)          // stained glass, primary action only
.interactive(_ isEnabled: Bool = true)  // press feedback: scale, bounce, glow

// Modifier — apply LAST, after padding/frame/font
func glassEffect(_ glass: Glass = .regular, in shape: some Shape = DefaultGlassEffectShape()) -> some View
// DefaultGlassEffectShape = Capsule
```

```swift
Text("Hello").padding().glassEffect()                                   // capsule, regular
Image(systemName: "mic").frame(width: 44, height: 44).glassEffect(.regular.interactive(), in: .circle)
Label("Send", systemImage: "paperplane").padding()
    .glassEffect(.regular.tint(.blue).interactive())                    // the one primary action
CardControls().glassEffect(.regular, in: .rect(cornerRadius: 24))      // large → rounded rect
NestedThing().glassEffect(.regular, in: ConcentricRectangle())         // concentric with container
```

Shapes: `.capsule` (default), `.circle`, `.ellipse`, `.rect(cornerRadius:)`,
`.rect(corners:isUniform:)`, `ConcentricRectangle()`, any `Shape`.

Corners are squircles by default:
- `.rect(cornerRadius:style:)` and `RoundedRectangle(cornerRadius:style:)` default to
  `RoundedCornerStyle.continuous` (verified in the SDK 27 docs). Pass `.circular` only to
  match round hardware-like shapes.
- UIKit `CALayer.cornerCurve` defaults to `.circular`. Set
  `layer.cornerCurve = .continuous` on any custom rounded view next to glass, or use
  `UICornerConfiguration`.
- Figma equivalent: corner smoothing 60 %. Apple's own UI kit uses it on every glass element
  (`anatomy.md` §2.7).

## 3. Containers, union, morphing

```swift
@Namespace private var ns
@State private var expanded = false

GlassEffectContainer(spacing: 40) {            // merge threshold in points
    HStack(spacing: 40) {
        Image(systemName: "pencil").frame(width: 80, height: 80)
            .glassEffect()
            .glassEffectID("pencil", in: ns)
        if expanded {
            Image(systemName: "eraser").frame(width: 80, height: 80)
                .glassEffect()
                .glassEffectID("eraser", in: ns)
                .glassEffectTransition(.matchedGeometry)   // default inside spacing
        }
    }
}
Button("Toggle") { withAnimation(.bouncy) { expanded.toggle() } }
    .buttonStyle(.glass)
```

Rules:
- One container per cluster of glass. It renders all shapes in one pass (performance) and
  is the only way glass shapes can blend/morph. "Glass cannot sample other glass."
- `spacing` > the stack's spacing ⇒ shapes are fused **at rest**. Equal ⇒ they fuse only
  when moved closer.
- `glassEffectID(_:in:)` + `withAnimation` ⇒ morph between hierarchy states.
- `GlassEffectTransition`: `.matchedGeometry` (default, morph into neighbour within
  spacing), `.materialize` (lensing ramps in/out; use when farther than spacing),
  `.identity`.
- `glassEffectUnion(id:namespace:)` ⇒ several views share **one** glass shape even at rest
  (Maps zoom +/−, segmented clusters, dynamically created items).
- Too many containers or loose glass views degrade performance — limit on-screen glass.

## 4. Buttons and controls

```swift
Button("Edit") { }.buttonStyle(.glass)                 // GlassButtonStyle
Button("Done") { }.buttonStyle(.glassProminent)        // tinted with accent colour
Button("Skip") { }.buttonStyle(.glass(.clear))         // iOS 26.1+: GlassButtonStyle(_ glass:)
```

Sliders and toggles turn their knob into a glass *lens* while dragged — free with system
controls; to reproduce it custom, see `motion.md` §2.2 (measured lens) and §3 (choreography).

## 5. Bars, toolbars, tabs, sidebars

```swift
TabView {
    Tab("Home", systemImage: "house") { Home() }
    Tab(role: .search) { Search() }                     // trailing search tab
}
.tabViewStyle(.sidebarAdaptable)                       // tab bar ↔ sidebar
.tabBarMinimizeBehavior(.onScrollDown)

.toolbar {
    ToolbarSpacer(.flexible)
    ToolbarItem { ShareLink(item: url) }
    ToolbarSpacer(.fixed)                              // splits shared glass backgrounds
    ToolbarItemGroup { FavoriteButton(); CollectionsMenu() }
}

.scrollEdgeEffectStyle(.soft, for: .top)               // .hard when pinned headers sit under the bar
.safeAreaBar(edge: .bottom) { CustomBar() }            // custom bar that gets the scroll edge effect
Image("hero").backgroundExtensionEffect()             // mirror+blur under sidebar/inspector
```

Don't mix text and icons inside one shared toolbar glass group. Hide whole
`ToolbarItem`s (`.hidden(_:)`), not their content, or an empty glass pill remains.

### New in SDK 27 (WWDC26)

- Liquid Glass responds to the system **transparency slider** automatically; no API needed.
- Custom glass marked `.interactive()` now also responds to the macOS pointer.
- `@Environment(\.appearsActive)` — dim custom glass content when the iPad/Mac window is
  inactive (system glass recedes automatically).
- `ToolbarOverflowMenu { … }` — items that always live in the overflow menu.
- Toolbar item `visibilityPriority` (high keeps an item visible when space is short) and a
  `topBarPinnedTrailing` placement (never hidden).
- `toolbarMinimizeBehavior(.onScrollDown, for: .navigationBar)` — nav bar recedes on scroll.
- A *prominent* tab role places e.g. a cart tab separately at the trailing edge.
- iPhone apps become resizable (iPhone mirroring, iPhone-on-iPad) — use size classes, not
  idiom, so glass layouts reflow.

(Exact spellings of the toolbar additions: check the SDK headers; the SwiftUI WWDC26
session demoed them by these names.)

## 6. UIKit

```swift
// Single element
let glass = UIGlassEffect(style: .regular)     // .regular | .clear
glass.isInteractive = true
glass.tintColor = .systemBlue                   // primary action only
let view = UIVisualEffectView(effect: glass)
view.cornerConfiguration = .capsule()           // or UICornerConfiguration with concentric radii
view.contentView.addSubview(label)

// Container: nest glass effect views inside one container view
let container = UIGlassContainerEffect()
container.spacing = 20
let containerView = UIVisualEffectView(effect: container)
containerView.contentView.addSubview(view)      // + more UIVisualEffectView(UIGlassEffect)

// Animate in/out: set effect inside UIView.animate — glass materializes
UIView.animate { view.effect = UIGlassEffect(style: .regular) }

// Buttons
var config = UIButton.Configuration.glass()     // .prominentGlass() .clearGlass() .prominentClearGlass()
```

Other: `UIScrollEdgeElementContainerInteraction` (custom bars get the scroll edge effect),
`UIBackgroundExtensionView`, `tabBarMinimizeBehavior = .onScrollDown`, `UISearchTab`,
`UIBarButtonItem.fixedSpace(_:)`. UIKit glass is opt-in per view; the SwiftUI and UIKit
containers don't merge with each other — keep a cluster in one framework.

## 7. AppKit

```swift
let glass = NSGlassEffectView()
glass.contentView = toolbarStack
glass.cornerRadius = 18
glass.style = .regular                // NSGlassEffectView.Style (.regular / .clear)
glass.tintColor = .controlAccentColor // optional
glass.effectIsInteractive = true      // pointer/press feedback
let container = NSGlassEffectContainerView()
container.spacing = 16
container.contentView = clusterView   // descendant NSGlassEffectViews merge
```

`NSButton` with `bezelStyle = .glass`, `NSBackgroundExtensionView`,
`NSSplitViewController(inspectorWithViewController:)`, `NSToolbarItem.space`.

## 7a. API inventory (every Liquid Glass object, one table)

Use it as a checklist when porting a design: each row is the native object and what a custom
renderer (Liquid Rust, the CSS preset) must reproduce. Rows marked † were announced at
WWDC25/26 sessions; check the exact spelling in the SDK headers before relying on it.

| Area | SwiftUI | UIKit | AppKit | Custom equivalent |
|---|---|---|---|---|
| Material | `Glass.regular / .clear / .identity`, `.tint(_:)`, `.interactive(_:)` | `UIGlassEffect(style: .regular / .clear)`, `.tintColor`, `.isInteractive` | `NSGlassEffectView.style`, `.tintColor`, `.effectIsInteractive`† | `Material::regular/clear/tinted/identity`; `.lg-glass`, `.lg-clear`, `.lg-tinted` |
| Apply | `glassEffect(_:in:)`, `DefaultGlassEffectShape` (capsule) | `UIVisualEffectView(effect:)` | `NSGlassEffectView.contentView` | `scene.add(Glass::new(…))` |
| Container / merge | `GlassEffectContainer(spacing:)` | `UIGlassContainerEffect` (`.spacing`) | `NSGlassEffectContainerView` (`.spacing`) | `add_container(spacing, z)` |
| Morph identity | `glassEffectID(_:in:)` + `@Namespace` | (set `effect` inside `UIView.animate`) | — | `morph_from` / `shrink_into` (one shape), `expand_from` (with neck) |
| Union at rest | `glassEffectUnion(id:namespace:)` | nested in one container | nested in one container | same container, spacing > gap |
| Transitions | `glassEffectTransition(.matchedGeometry / .materialize / .identity)` | — | — | morph vs `add` / `remove` (lensing ramps) |
| Buttons | `.buttonStyle(.glass)`, `.glassProminent`, `.glass(.clear)`; `.buttonBorderShape(.capsule / .circle / .roundedRectangle)`; `.controlSize(.extraLarge)`† | `UIButton.Configuration.glass() / .prominentGlass() / .clearGlass() / .prominentClearGlass()` | `NSButton.BezelStyle.glass` | `.interactive()`; `.lg-button` |
| Toggle / Slider | system `Toggle`, `Slider` (knob turns into a lens while held); `Slider` tick marks / neutral value† | `UISwitch`, `UISlider` (track configuration / thumbless style†) | `NSSwitch`, `NSSlider` | `.lens()` glass; `.lg-switch`, `.lg-slider` + `use:refraction` |
| Segmented | `Picker(.segmented)` | `UISegmentedControl` | `NSSegmentedControl` | `.lg-segmented` |
| Menus, popovers | `Menu`, `.popover`, `.presentationCompactAdaptation(.popover)` | `UIMenu`, `UIPopoverPresentationController` | `NSMenu`, `NSPopover` | `morph_from` the trigger; `.lg-pop` |
| Sheets | `.sheet` + `.presentationDetents([.medium, .large])` (partial = inset glass, large = opaque); `.presentationBackground(_:)` replaces the glass | `UISheetPresentationController.detents` | sheets on windows | `.lg-sheet` / `.lg-sheet-full` |
| Zoom from source | `.matchedTransitionSource(id:in:)` on the button + `.navigationTransition(.zoom(sourceID:in:))` on the sheet or pushed view (the sheet grows out of the toolbar button) | `preferredTransition = .zoom { … }` | — | `morph_from` + content blur-in |
| Toolbars | `ToolbarItem`, `ToolbarItemGroup`, `ToolbarSpacer(.fixed / .flexible)`, `.sharedBackgroundVisibility(.hidden)`†, `.badge`† | `UIBarButtonItem.hidesSharedBackground`†, `.style = .prominent`†, `.fixedSpace(_:)` | `NSToolbarItem.space` | one container per group |
| Tab bar | `TabView`, `Tab(role: .search)`, `.tabViewStyle(.sidebarAdaptable)`, `.tabBarMinimizeBehavior(.onScrollDown)`, `.tabViewBottomAccessory { }`† | `UITabBarController.tabBarMinimizeBehavior`, `UISearchTab`, bottom accessory† | — | `.lg-bar` |
| Search | `.searchable`, `.searchToolbarBehavior(.minimize)`† | `UISearchTab`, `UISearchController` | `NSSearchToolbarItem` | capsule `.lg-bar` |
| Scroll edge | `.scrollEdgeEffectStyle(.soft / .hard, for:)`, `.scrollEdgeEffectHidden(_:for:)`†, `.safeAreaBar(edge:)` | `UIScrollView.topEdgeEffect.style`†, `UIScrollEdgeElementContainerInteraction` | `NSScrollView` edge effect | gradient + blur band under bars |
| Background extension | `.backgroundExtensionEffect()` | `UIBackgroundExtensionView` | `NSBackgroundExtensionView` | mirror + blur the hero under the sidebar |
| Corners | `ConcentricRectangle`, `.rect(corners: .concentric)`†, `ContainerRelativeShape`, `.containerShape(_:)` | `UICornerConfiguration`, `UICornerRadius.containerConcentric(minimum:)` | `NSView` corner config† | `.lg-squircle` (`corner-shape: superellipse(1.478)`, radius × 1.3168) |
| Navigation titles | `.navigationSubtitle(_:)` | `UINavigationItem.subtitle`, `.largeSubtitle`† | `NSWindow.subtitle` | — |
| Window state | `@Environment(\.appearsActive)` | — | — | dim custom glass when inactive |
| Widgets / icons | `.widgetAccentedRenderingMode(_:)`, `WidgetRenderingMode.accented`; Icon Composer layers | — | — | — |
| visionOS | `glassBackgroundEffect(displayMode:)` | — | — | — |
| Accessibility | Reduce Transparency, Increase Contrast, Reduce Motion, the transparency slider (automatic) | same | same | `Appearance { reduce_transparency, increase_contrast, reduce_motion, transparency }`; CSS media queries |

## 8. visionOS

Windows use the system *glass* background; `glassBackgroundEffect(displayMode:)` with
`.automatic`, `.feathered(padding:softEdgeRadius:)`, `.plate` styles. Don't add Liquid Glass
on top of window glass.

## 9. Icons (Icon Composer)

Design in layers (foreground / middle / background), solid filled overlapping shapes; let
the system add reflection, refraction, shadow, blur, highlights. Provide default, dark,
clear and tinted appearances. SDK 27 Icon Composer: multiple glass layers, per-layer
refraction annotations, preview of the fallback on older OSes.

## 10. Migration checklist

- [ ] Build with Xcode 27; remove `UIDesignRequiresCompatibility` when ready.
- [ ] Delete custom bar/sidebar/sheet backgrounds and old `.ultraThinMaterial` on controls.
- [ ] Group neighbouring custom glass in `GlassEffectContainer`.
- [ ] `.interactive()` on custom tappable glass only.
- [ ] One tinted primary action per screen.
- [ ] No glass on content (lists, cards); no glass on glass.
- [ ] Clear variant only over media, with 35 % dimming if bright.
- [ ] Test: light/dark, Reduce Transparency, Increase Contrast, Reduce Motion, transparency
      slider at both ends and middle, inactive window, iPhone resizing.
- [ ] Test on device — specular motion response doesn't run in Simulator.
- [ ] Games: `glassEffect(.regular, …)` only in menus; disable during gameplay.

## 11. Common mistakes

| Mistake | Fix |
|---|---|
| `.glassEffect()` before `.padding()` | apply last |
| Adjacent glass without a container | wrap in `GlassEffectContainer` |
| Morph doesn't happen | same container + same namespace + `withAnimation` |
| Glass button with `.background(Color.blue)` | `.glassEffect(.regular.tint(.blue))` or `.glassProminent` |
| Glass sub-buttons inside a glass card | fills/vibrancy inside, glass only outside |
| `.clear` over a white settings screen | `.regular` |
| Hiding a toolbar item's content | hide the item |
