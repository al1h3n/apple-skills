# Native Apple API: SwiftUI, UIKit, AppKit

On Apple platforms **always use the system API**. Gets the real compositor (quarter-res backdrop, SDF union, luma tracking, HDR highlights), accessibility, transparency slider, future refinements. Never hand-roll glass shaders in a native Apple app.

Verified against developer.apple.com (SDK 27). `(26.1)`, `(27)` = first SDK with that API; unmarked = 26. Rows marked † = announced in a session; check spelling in SDK headers. Deploying below 26 → `if #available(iOS 26, macOS 26, *)`.

Custom-renderer equivalents: `liquid-rust.md`, `bits-ui.md`.

## 1. Free adoption

Rebuild with current Xcode: `NavigationStack`, `NavigationSplitView`, `TabView`, toolbars, sheets, popovers, menus, `Button` / `Toggle` / `Slider` / `Picker` / `Stepper` / `TextField` adopt the material. **Remove custom backgrounds from bars and split views**: they block system glass and the scroll edge effect.

`UIDesignRequiresCompatibility = YES` (Info.plist) = pre-26 look, going away. Plan for glass.

## 2. SwiftUI core

```swift
// struct Glass
.regular                                // default, adaptive
.clear                                  // media overlays only
.identity                               // no effect: transition endpoint, conditional off
.tint(_ color: Color?)                  // primary action only
.interactive(_ isEnabled: Bool = true)  // press feedback; also macOS pointer (27)

// Apply LAST, after padding / frame / font
func glassEffect(_ glass: Glass = .regular, in shape: some Shape = DefaultGlassEffectShape()) -> some View
// DefaultGlassEffectShape = Capsule
```

```swift
Text("Hello").padding().glassEffect()
Image(systemName: "mic").frame(width: 44, height: 44).glassEffect(.regular.interactive(), in: .circle)
Label("Send", systemImage: "paperplane").padding()
    .glassEffect(.regular.tint(.blue).interactive())                 // the one primary action
CardControls().glassEffect(.regular, in: .rect(cornerRadius: 24))
NestedThing().glassEffect(.regular, in: ConcentricRectangle())      // concentric with container
```

Shapes: `.capsule`, `.circle`, `.ellipse`, `.rect(cornerRadius:)`, `.rect(corners:isUniform:)`, `ConcentricRectangle()`, any `Shape`.

Corners:
- `.rect(cornerRadius:style:)`, `RoundedRectangle` default to `.continuous`. `.circular` only for hardware-like shapes.
- UIKit `CALayer.cornerCurve` defaults to `.circular` → set `.continuous` on any custom rounded view next to glass, or use `UICornerConfiguration`.

## 3. Containers, union, morph

```swift
@Namespace private var ns
@State private var expanded = false

GlassEffectContainer(spacing: 40) {            // merge threshold, pt
    HStack(spacing: 40) {
        Image(systemName: "pencil").frame(width: 80, height: 80)
            .glassEffect()
            .glassEffectID("pencil", in: ns)
        if expanded {
            Image(systemName: "eraser").frame(width: 80, height: 80)
                .glassEffect()
                .glassEffectID("eraser", in: ns)
                .glassEffectTransition(.matchedGeometry)
        }
    }
}
Button("Toggle") { withAnimation(.smooth) { expanded.toggle() } }
    .buttonStyle(.glass)
```

- One container per cluster. One pass; only way shapes blend or morph. Glass cannot sample other glass.
- `spacing` > stack spacing → fused at rest. Equal → fuse only when moved closer.
- `glassEffectID(_:in:)` + `withAnimation` → morph between hierarchy states.
- `GlassEffectTransition`: `.matchedGeometry` (default, within spacing), `.materialize` (lensing ramps; farther than spacing), `.identity`.
- `glassEffectUnion(id:namespace:)` → several views, **one** shape at rest (zoom +/−, clusters).
- Many containers or loose glass views cost frames. Limit on-screen glass.

## 4. Buttons, controls

```swift
Button("Edit") { }.buttonStyle(.glass)
Button("Done") { }.buttonStyle(.glassProminent)        // tinted with accent
Button("Skip") { }.buttonStyle(.glass(.clear))         // (26.1) GlassButtonStyle(_ glass:)
```

System `Toggle` / `Slider`: knob becomes a lens while held, free. Custom: `anatomy.md` §4 lens, `motion.md` §3.

## 5. Bars, tabs, sidebars

```swift
TabView {
    Tab("Home", systemImage: "house") { Home() }
    Tab(role: .search) { Search() }                     // separate trailing search
}
.tabViewStyle(.sidebarAdaptable)
.tabBarMinimizeBehavior(.onScrollDown)

.toolbar {
    ToolbarSpacer(.flexible)
    ToolbarItem { ShareLink(item: url) }
    ToolbarSpacer(.fixed)                              // splits a shared glass background
    ToolbarItemGroup { FavoriteButton(); CollectionsMenu() }
}

.scrollEdgeEffectStyle(.soft, for: .top)               // .hard when pinned headers sit under the bar
.safeAreaBar(edge: .bottom) { CustomBar() }            // custom bar gets the scroll edge effect
Image("hero").backgroundExtensionEffect()              // mirror + blur under sidebar / inspector
```

- No text + icons mixed in one shared toolbar group.
- Hide whole `ToolbarItem`s (`.hidden(_:)`), not their content: else an empty pill stays.

SDK 27:
- Transparency slider honoured automatically.
- `@Environment(\.appearsActive)`: dim custom glass content when the window is inactive (system glass flattens by itself).
- `ToolbarOverflowMenu { … }`: items that always live in overflow.
- Toolbar item `visibilityPriority`†, placement `topBarPinnedTrailing`† (never hidden).
- `toolbarMinimizeBehavior(.onScrollDown, for: .navigationBar)`†.
- Prominent tab role†: e.g. a cart tab apart at the trailing edge.
- iPhone apps resizable (mirroring, iPhone-on-iPad): size classes, not idiom.

## 6. UIKit

```swift
let glass = UIGlassEffect(style: .regular)     // .regular | .clear
glass.isInteractive = true
glass.tintColor = .systemBlue                   // primary action only
let view = UIVisualEffectView(effect: glass)
view.cornerConfiguration = .capsule()
view.contentView.addSubview(label)

let container = UIGlassContainerEffect()
container.spacing = 20
let containerView = UIVisualEffectView(effect: container)
containerView.contentView.addSubview(view)      // + more glass effect views

UIView.animate { view.effect = UIGlassEffect(style: .regular) }   // materializes

var config = UIButton.Configuration.glass()     // .prominentGlass() .clearGlass() .prominentClearGlass()
```

Also: `UIScrollEdgeElementContainerInteraction`, `UIBackgroundExtensionView`, `tabBarMinimizeBehavior = .onScrollDown`, `UISearchTab`, `UIBarButtonItem.fixedSpace(_:)`. Opt-in per view. SwiftUI and UIKit containers do not merge with each other: one cluster, one framework.

## 7. AppKit

```swift
let glass = NSGlassEffectView()
glass.contentView = toolbarStack
glass.cornerRadius = 18
glass.style = .regular                // .regular | .clear
glass.tintColor = .controlAccentColor
glass.effectIsInteractive = true
let container = NSGlassEffectContainerView()
container.spacing = 16
container.contentView = clusterView   // descendant glass views merge
```

`NSButton.bezelStyle = .glass`, `NSBackgroundExtensionView`, `NSSplitViewController(inspectorWithViewController:)`, `NSToolbarItem.space`.

## 8. Inventory

| Area | SwiftUI | UIKit | AppKit |
|---|---|---|---|
| Material | `Glass.regular / .clear / .identity`, `.tint`, `.interactive` | `UIGlassEffect(style:)`, `.tintColor`, `.isInteractive` | `NSGlassEffectView.style`, `.tintColor`, `.effectIsInteractive`† |
| Apply | `glassEffect(_:in:)` | `UIVisualEffectView(effect:)` | `NSGlassEffectView.contentView` |
| Container | `GlassEffectContainer(spacing:)` | `UIGlassContainerEffect` | `NSGlassEffectContainerView` |
| Morph | `glassEffectID(_:in:)` + `@Namespace` | set `effect` in `UIView.animate` | none |
| Union | `glassEffectUnion(id:namespace:)` | nested in one container | same |
| Transition | `glassEffectTransition(…)` | none | none |
| Buttons | `.glass`, `.glassProminent`, `.glass(.clear)` (26.1); `.buttonBorderShape`; `.controlSize(.extraLarge)`† | `UIButton.Configuration.glass()` family | `NSButton.BezelStyle.glass` |
| Toggle, slider | `Toggle`, `Slider` (tick marks, neutral value†) | `UISwitch`, `UISlider` | `NSSwitch`, `NSSlider` |
| Segmented | `Picker(.segmented)` | `UISegmentedControl` | `NSSegmentedControl` |
| Menu, popover | `Menu`, `.popover`, `.presentationCompactAdaptation(.popover)` | `UIMenu`, `UIPopoverPresentationController` | `NSMenu`, `NSPopover` |
| Sheet | `.sheet` + `.presentationDetents` (partial = inset glass, large = opaque); `.presentationBackground` replaces the glass | `UISheetPresentationController.detents` | window sheets |
| Zoom from source | `.matchedTransitionSource(id:in:)` + `.navigationTransition(.zoom(sourceID:in:))` | `preferredTransition = .zoom { … }` | none |
| Toolbar | `ToolbarItem(Group)`, `ToolbarSpacer`, `.sharedBackgroundVisibility(.hidden)`†, `.badge`†, `ToolbarOverflowMenu` (27) | `UIBarButtonItem.hidesSharedBackground`†, `.style = .prominent`†, `.fixedSpace` | `NSToolbarItem.space` |
| Tab bar | `TabView`, `Tab(role: .search)`, `.sidebarAdaptable`, `.tabBarMinimizeBehavior`, `.tabViewBottomAccessory`† | `UITabBarController.tabBarMinimizeBehavior`, `UISearchTab` | none |
| Search | `.searchable`, `.searchToolbarBehavior(.minimize)`† | `UISearchController`, `UISearchTab` | `NSSearchToolbarItem` |
| Scroll edge | `.scrollEdgeEffectStyle`, `.scrollEdgeEffectHidden`†, `.safeAreaBar` | `UIScrollView.topEdgeEffect.style`†, `UIScrollEdgeElementContainerInteraction` | `NSScrollView` edge effect |
| Background extension | `.backgroundExtensionEffect()` | `UIBackgroundExtensionView` | `NSBackgroundExtensionView` |
| Corners | `ConcentricRectangle`, `.rect(corners: .concentric)`†, `ContainerRelativeShape`, `.containerShape` | `UICornerConfiguration`, `UICornerRadius.containerConcentric(minimum:)` | corner config† |
| Titles | `.navigationSubtitle` | `UINavigationItem.subtitle`, `.largeSubtitle`† | `NSWindow.subtitle` |
| Window state | `@Environment(\.appearsActive)` (27) | none | none |
| Widgets | `.widgetAccentedRenderingMode`, `WidgetRenderingMode.accented` | none | none |
| visionOS | `glassBackgroundEffect(displayMode:)`: `.automatic`, `.feathered(padding:softEdgeRadius:)`, `.plate`. No Liquid Glass on window glass | none | none |
| Accessibility | Reduce Transparency, Increase Contrast, Reduce Motion, slider: automatic | same | same |

Icons: Icon Composer (`icons.md`).

## 9. Migration checklist

- [ ] Current Xcode; drop `UIDesignRequiresCompatibility`.
- [ ] Delete custom bar / sidebar / sheet backgrounds and `.ultraThinMaterial` on controls.
- [ ] Neighbouring custom glass → one `GlassEffectContainer`.
- [ ] `.interactive()` only on tappable custom glass.
- [ ] One tinted primary action per screen.
- [ ] No glass on content; no glass on glass.
- [ ] Clear only over media, 35 % dim if bright.
- [ ] Test: light / dark, Reduce Transparency, Increase Contrast, Reduce Motion, slider at both ends + middle, inactive window, resized iPhone.
- [ ] Test on device: Simulator skips motion-driven highlights.
- [ ] Games: glass in menus only.

## 10. Common mistakes

| Mistake | Fix |
|---|---|
| `.glassEffect()` before `.padding()` | apply last |
| Adjacent glass, no container | `GlassEffectContainer` |
| Morph does not happen | same container + same namespace + `withAnimation` |
| `.background(Color.blue)` on a glass button | `.glassProminent` or `.tint(.blue)` |
| Glass sub-buttons in a glass card | fills + vibrancy inside |
| `.clear` over a plain screen | `.regular` |
| Hiding a toolbar item's content | hide the item |
