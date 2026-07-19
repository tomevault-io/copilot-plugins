## svelte-devtools

> A Vite plugin + overlay UI for inspecting Svelte 5 / SvelteKit apps during development.

# Svelte DevTools

A Vite plugin + overlay UI for inspecting Svelte 5 / SvelteKit apps during development.

## Monorepo structure

```
packages/
  overlay/       — Svelte 5 panel UI (pre-built to dist/overlay.js)
  vite-plugin/   — Vite plugin that serves virtual modules
  kit/           — SvelteKit handle hook for HTML injection
playground/      — SvelteKit dev sandbox
```

## Commands

```bash
npm run setup          # install deps + build overlay (run once after clone)
npm run build:overlay  # rebuild the overlay UI bundle
npm run dev            # start playground at localhost:5173
npm run dev:overlay    # watch-build the overlay (use alongside npm run dev)
```

## Architecture

### Two virtual modules (packages/vite-plugin)

- `virtual:svelte-devtools` — bootstrap module: initializes `window.__sdt`, subscribes to `$app/stores` on SvelteKit
- `virtual:svelte-devtools-overlay` — serves the pre-built `packages/overlay/dist/overlay.js` verbatim

The vite plugin watches `overlay.js` on disk. When `npm run build:overlay` completes it invalidates the Vite module graph and triggers a full page reload automatically.

### Why `packages/kit` exists

SvelteKit never calls `server.transformIndexHtml`. The only way to inject a `<script>` into SSR-rendered HTML is via `resolve(event, { transformPageChunk })` in a SvelteKit handle hook. The kit package exports `svelteDevtoolsHandle` for this.

Wire it up in `hooks.server.ts`:
```ts
import { sequence } from '@sveltejs/kit/hooks';
import { svelteDevtoolsHandle } from '../../packages/kit/src/index.ts';
export const handle = sequence(svelteDevtoolsHandle);
```

### Overlay build (packages/overlay)

- Built with its own `vite.config.ts` as an ESM library (`formats: ['es']`)
- `emitCss: false` — all styles live in `src/styles.ts` as a JS string, injected as a `<style id="sdt-styles">` tag at runtime
- All CSS class names are `sdt-` prefixed to avoid collisions
- Entry point `src/app.ts` mounts `Overlay.svelte` into `#__svelte-devtools__`

### Communication pattern

```
window.__sdt = { page, navigating, componentState }
CustomEvent '__sdt:update'  →  tabs re-read window.__sdt
```

SvelteKit store subscriptions fire `__sdt:update` automatically. To expose local runes state from a component:

```ts
$effect(() => {
  (window as any).__sdt.componentState = { count, name };
  window.dispatchEvent(new CustomEvent('__sdt:update', { detail: { type: 'state' } }));
});
```

### Panel layout

The panel has two positions, toggled by the position button in the header (persisted to `localStorage`):

- **Bottom drawer** (default) — full header with logo text and labeled tabs; height resizable by dragging the top edge
- **Right drawer** — compact icon-only header (logo text + tab labels hidden to fit); width resizable by dragging the left edge

The header adapts automatically: when `position === 'right'`, `Panel.svelte` hides the "Svelte DevTools" wordmark and tab label text, showing only the SVG logo and tab icons. Toggle between modes with the layout icon in the header, or press ⇧⌥D to open/close.

## CSS rules

- No `<style>` blocks in overlay components — Svelte's CSS extraction breaks library builds
- All styles go in `packages/overlay/src/styles.ts` as the exported `CSS` string
- Use `sdt-` prefix on every class name
- The CSS reset uses two separate selectors to avoid specificity conflicts:
  - `#__svelte-devtools__ * { box-sizing: border-box }` — ID selector for box-sizing only
  - `.sdt-panel * { margin: 0; padding: 0 }` — class selector for spacing reset, so content rules (also class-level) can override via cascade order

## Svelte 5 notes

- Use explicit self-import instead of `<svelte:self>` (deprecated in Svelte 5): `import JsonNode from './JsonNode.svelte'`
- Wrap prop-derived initial state in `untrack()` to avoid reactive capture warnings: `let expanded = $state(untrack(() => depth < 2))`
- Button elements need `border: none` explicitly — browser default `border: 2px outset white` shows through on dark backgrounds

## Key files

| File | Purpose |
|---|---|
| `packages/vite-plugin/src/index.ts` | Vite plugin — virtual modules + overlay watcher |
| `packages/kit/src/index.ts` | SvelteKit `transformPageChunk` handle |
| `packages/overlay/src/app.ts` | Overlay entry — mounts Svelte component |
| `packages/overlay/src/styles.ts` | All CSS as a JS string |
| `packages/overlay/src/Overlay.svelte` | Toggle button, keyboard shortcut (⇧⌥D), panel open/position/size state |
| `packages/overlay/src/Panel.svelte` | Adaptive header (bottom vs right mode), tab bar, resize handle, tab routing |
| `packages/overlay/src/tabs/RoutesTab.svelte` | SvelteKit page/route inspector |
| `packages/overlay/src/tabs/ComponentsTab.svelte` | DOM tree with hover-to-highlight |
| `packages/overlay/src/tabs/StateTab.svelte` | Component state + SvelteKit page store |
| `packages/overlay/src/JsonNode.svelte` | Recursive JSON tree viewer |
| `playground/src/hooks.server.ts` | Wires up `svelteDevtoolsHandle` |
| `playground/vite.config.ts` | Adds `svelteDevtools()` plugin |
| `.Codex/launch.json` | Preview server config for Codex |

---
> Source: [chrislentz/svelte-devtools](https://github.com/chrislentz/svelte-devtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
