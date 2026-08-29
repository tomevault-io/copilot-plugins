## cms-web-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server with HMR
npm run build     # Type-check (tsc -b) then bundle (vite build)
npm run lint      # Run ESLint
npm run preview   # Serve the production build locally
```

There are no automated tests configured in this project.

## Architecture

### Entry & Routing

`src/main.tsx` wraps the app in `BrowserRouter`. `src/App.tsx` is the route shell — currently only `/dashboard` exists; everything else redirects there.

### Page Layout Pattern

Every page is rendered inside `RenderPage` (a layout shell in `src/components/`):

```
App → RenderPage → sections/Header (sticky) + sections/Content (body)
                          ↓
                    <children> (page component)
```

`RenderPage` manages the sidebar `Menu` drawer open/closed state. New pages follow the pattern: add a `<Route>` in `App.tsx` pointing to a page component, wrapped in `RenderPage`.

Pages live in `src/pages/<PageName>/index.tsx`.

### Theming

**Always** import colors, fonts, and breakpoints from `src/theme.ts` — never use inline hex values. Key exports:

- `Colors.*` — full color palette (primary brand: `Colors.main = "#fb5103"`)
- `Fonts.*` — `Fonts.main = "Fira Sans"`, `Fonts.secondary = "Inter"`, `Fonts.buttonFont = "Outfit"`
- `Breakpoints.lg` — `"@media (max-width: 1200px)"` (main responsive breakpoint)
- `BarChartPalette` — array of colors for chart series

All styling is done with MUI's `sx` prop or `@emotion/styled`. No CSS modules or global stylesheets.

### Timeline System (Two-File Split)

The timeline is split across two files for separation of concerns:

- **`src/components/TimeLine.tsx`** — React shell. Uses `forwardRef` to expose a `TimeLineHandle` imperative API (`setCategory`, `on/off/once/emit`, `renderer`). Three canvas elements (`timesRef`, `mainRef`, `markerRef`) are created in JSX and passed to the renderer. The renderer is instantiated in a **mount-only `useEffect`** (empty dep array) and destroyed on unmount. Snapshot data flows in via `renderer.invalidateSnapshot(snapshot)` in a separate effect.

- **`src/components/TimelineRenderer.ts`** — Canvas drawing engine (ported from `src/migrate/component.js`). Uses private class fields (`#field` syntax). Exposes a rich API: `init()`, `destroy()`, `invalidateSnapshot()`, `addTrack()`, `cut()`, `patch()`, `addSession()`, `moveToFirst/Last/Forward/Backward()`, `togglePlay()`, `setCategory()`, etc.

- **`src/migrate/moment.ts`** — Dayjs shim that mirrors the moment.js API. Use this instead of importing moment.js (not installed).
- **`src/migrate/ComponentComplement.ts`** — Event-emitter base class used by the renderer.

### CameraLayout

`src/components/CameraLayout.tsx` renders 1–16 cameras in an adaptive grid. The `getRowDistribution()` function determines row layout automatically. Each camera slot uses a right-click `EventMenu` for context actions.

### TypeScript Config

Strict mode is on with `noUnusedLocals` and `noUnusedParameters`. Target is `ES2022` (private class fields `#field` are valid). Run `tsc --noEmit` to type-check without building.

---
> Source: [lugonzalezrebiz/cms-web-react](https://github.com/lugonzalezrebiz/cms-web-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
