## aonsoku-reborn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Aonsoku is a dual-target music streaming client (web + Electron desktop) for Navidrome/Subsonic servers. It supports synchronized lyrics, podcasts, radio streaming, and Discord RPC.

## Commands

```bash
# Development
pnpm run dev              # Web app dev server
pnpm run electron:dev     # Electron desktop dev

# Build
pnpm run build            # Production web build (tsc + vite)
pnpm run electron:build   # Electron renderer/main/preload build
pnpm run make             # Create distributable packages

# Lint & Format (Biome)
pnpm run lint             # Check only
pnpm run lint:fix         # Auto-fix issues
pnpm run lint:format      # Format files

# Tests (Cypress component tests)
pnpm run test             # Headless run
pnpm run cy:open          # Interactive test runner
```

## Architecture

### Dual-Target Build

- **Web**: Vite (`vite.config.ts`) builds `src/` as a standard SPA
- **Electron**: `electron.vite.config.ts` builds three separate processes: `electron/main`, `electron/preload`, `electron/renderer` (which re-uses `src/`)
- Platform detection via `src/utils/desktop.ts`

### State Management

Two complementary systems:
- **Zustand** (`src/store/`) — client state with `immer`, `persist`, `subscribeWithSelector` middleware. The player store (`player.store.ts`) is the most complex (~55KB). Persistent data (queue, song lists) goes through `idb.ts` (IndexedDB).
- **TanStack React Query** (`src/queries/`) — server state, caching, and invalidation. Query client configured in `src/lib/queryClient.ts` (refetch-on-window-focus disabled).

### Data Flow

```
Subsonic/Navidrome server
  → src/api/httpClient.ts (base HTTP)
  → src/service/*.ts (domain API wrappers)
  → src/queries/*.ts (React Query hooks)
  → React components
```

### Routing

React Router v6 with hash routing. Route constants live in `src/routes/routesList.ts`. Auth is enforced via loaders (`protectedLoader.ts`, `loginLoader.ts`) — always use named route constants from `ROUTES` rather than hardcoded strings.

### Component Patterns

- Radix UI primitives + shadcn/ui copy-paste pattern (full ownership of component code)
- `src/app/components/` — reusable UI components
- `src/app/pages/` — route-level page components
- `src/app/tables/` — data table implementations
- Observers (theme, lang, media session) are mounted at the root in `App.tsx`

### Styling

Tailwind CSS with CSS custom properties for layout dimensions (`--header-height`, `--sidebar-width`, `--player-height`). Theme variables (HSL colors for dark/light) are in `src/themes.css`. Dark mode is class-based.

### Code Splitting

Manual chunk strategy defined in `src/manual-chunks.ts` — update this when adding large new dependencies.

## Tooling Notes

- **Package manager**: pnpm only
- **Linter/formatter**: Biome 2.0.6 — double quotes, trailing commas, 80-char line width, no unused vars/imports
- **TypeScript**: strict mode, path alias `@/` → `src/`
- **Git hooks**: Husky is configured (`.husky/`)

---
> Source: [realtvop/aonsoku-reborn](https://github.com/realtvop/aonsoku-reborn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
