# Svelte and SolidJS: the component kits

Two packages with the same components, props and class names over one framework-free core,
so an app can move between Svelte 5 and SolidJS without the glass changing:

| Package | Framework | On | Install (git; built `dist/` committed) |
|---|---|---|---|
| `bits-ui-liquid-glass` | Svelte 5.33+ | Bits UI 2.21+ | `pnpm add github:al1h3n/BitsUILiquidGlass bits-ui` |
| `solid-liquid-glass` | SolidJS 1.9+ | Kobalte 0.13.14+ | `pnpm add github:al1h3n/SolidLiquidGlass @kobalte/core` |
| `liquid-rust-web` (both use it) | none | liquid-rust 0.3.0 wasm | `github:al1h3n/liquid-rust#path:/js` |

Icons: Lucide (`@lucide/svelte`, `lucide-solid`). Live gallery with both codes: the
liquid-rust showcase (`components.html`).

## Setup

```svelte
<script lang="ts">
  import 'bits-ui-liquid-glass/glass.css';
  import { GlassLayer, GlassGroup, Button, Switch, Menu, MenuItem } from 'bits-ui-liquid-glass';
  import Plus from '@lucide/svelte/icons/plus';
  let wifi = $state(true);
</script>

<GlassLayer backdrop="/wallpaper.jpg" style="height: 100dvh">
  <GlassGroup spacing={12}>
    <Button>Share</Button>
    <Menu>
      {#snippet trigger(props)}<Button {...props} symbol aria-label="Add"><Plus /></Button>{/snippet}
      <MenuItem>New Note</MenuItem>
    </Menu>
  </GlassGroup>
  <Switch bind:checked={wifi} aria-label="Wi-Fi" />
</GlassLayer>
```

SolidJS: same names; `checked` + `onCheckedChange` instead of `bind:checked`, `trigger={(props) => …}`
instead of a snippet, `ref` on `GlassLayer` receives the layer.

Vite: `optimizeDeps: { exclude: ['liquid-rust-web'] }`. Local checkouts: a git-ignored
`pnpm-workspace.yaml` with `overrides: { liquid-rust-web: link:../LiquidRust/js }`.

## `GlassLayer`

One backdrop, all glass over it. `backdrop` (address, `<img>` / `<video>` / `<canvas>`, or a
painter), `scheme` (`auto` | `light` | `dark`), `transparency` (0 ultra clear … 1 tinted),
`renderer` (`auto` | `css`), `legacy` (iOS 26 motion), `reduceTransparency` /
`reduceMotion` / `increaseContrast` (`system` | `on` | `off`), `zoom` / `blur` / `dim` (the
backdrop stepping back; glide), `adaptToContent`, `bind:layer`. The root gets
`data-lg-renderer`, `-scheme`, `-transparency`, `-contrast`, `-motion`.

Engine when WebGPU and a device exist; the CSS renderer otherwise (`layer.failure.reason`:
`unsupported`, `failed`, `off`). Nothing else changes for the components.

## Components (Apple → kit)

| Apple | Component |
|---|---|
| `glassEffect`, `GlassEffectContainer` | `Glass` (`kind` control / panel / clear / dock, `tint`, `radius`, `interactive`, `from`), `GlassGroup` (`spacing`), `Plain` |
| Buttons | `Button` (`variant` glass / prominent / clear / filled / gray / plain, `size`, `symbol`, `destructive`) |
| Toggle, Slider, Stepper | `Switch` (`mac`), `Slider`, `Stepper` |
| Segmented, pop-up button | `Segmented` (`glass`, `size`), `PopupButton` (`glass`, `size="mac"`) |
| Checkbox, radio group | `Checkbox` (`indeterminate`), `RadioList` |
| Text and search fields | `TextField` (`hint`, `error`), `SearchField` |
| Progress, page control, date picker | `Progress`, `PageControl`, `Calendar` |
| Tab bar, toolbar, sidebar, Dock | `TabBar` + `Tabs.Content`, `Toolbar` / `ToolbarGroup` / `ToolbarButton`, `Sidebar` (`size="mac"`), `Dock` (`size="mac"`, `magnify`) |
| Menus | `Menu` + `MenuItem` + `MenuSeparator` (one shape from its trigger), `ContextMenu`, `EditMenu`, `MenuBar` |
| Popover, tooltip | `Popover`, `Tooltip` |
| Alert, action sheet, sheet | `Alert` (`size="mac"`, `icon`), `ActionSheet`, `Sheet` (`detents`, drag, fling) |
| Notification | `Banner` (`size="mac"`, `duration`) |
| Lists | `List` (`heading`, `level`, `note`), `ListRow`, `SwipeRow` (`leading`, `trailing`) |
| Content | `EmptyState`, `Widget`, `WeatherWidget`, `ScrollEdge` (`look` soft / hard) |
| Control Center | `ControlModule` (`size` small / wide / large, `tint`) |
| Mac window | `Window` (`toolbar` snippet), `WindowControls`; `genie(window, dockIconRect, 'in' / 'out')` minimizes into the Dock |

## Building your own

Svelte: `{@attach glass(useLayer(), () => ({ kind: 'control', role: 'control' }))}`;
`useGlass`, `useSelection`, `useSurface`, `useLevel` / `provideLevel`, `useGroup`. SolidJS:
`createGlass(() => element, () => options)`, `createSelection`, `createSurface`. Plain JS:
`createLayer`, `glass` from `liquid-rust-web` (`webgpu.md` §2). Surfaces that open over the
layer render into its top element so they stack above every glass.

## Gotchas

- **Svelte effects must `untrack` calls into the core.** The reactive layer's getters are
  read inside `configure` / `setBackdrop`; tracking them loops (`effect_update_depth_exceeded`).
- **Menus choose on release.** The press that opens a menu never picks a row
  (`armAfterRelease`); keep it if you build your own surface.
- **Keep closing content mounted** (`forceMount` / Kobalte `forceMount`) so a surface folds
  back into its control instead of vanishing.
- **Windows reports Reduce Transparency** whenever its Transparency effects are off: the glass
  is frosted. Offer `reduceTransparency="off"` with a word of why.
- **The engine refracts the backdrop, not HTML.** Put the picture in `backdrop`.
- **Hidden inputs (Kobalte).** Tests click the visible control and assert through its role.

## Testing

Playwright: the installed Chrome (`channel: 'chrome'`, `--enable-unsafe-webgpu --enable-gpu
--ignore-gpu-blocklist`) runs the engine; a `css` project forces `renderer="css"`; Playwright's
headless shell has no WebGPU device and proves the fallback. axe on the stage with colour
contrast excluded inside glass. Both kits: 57 tests each (engine, CSS, fallback).
