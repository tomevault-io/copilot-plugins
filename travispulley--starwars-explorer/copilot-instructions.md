## starwars-explorer

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Overview

Star Wars Explorer - A demo frontend built with Bun + React 19 + Zustand + Ky + Tailwind 4 + shadcn/ui. Fetches data from the Star Wars API (SWAPI) at https://swapi.py4e.com/api/.

## Commands

```bash
bun install          # Install dependencies
bun dev              # Dev server with HMR at http://localhost:3000
bun start            # Production server
bun run build.ts     # Production build to dist/
```

## Architecture

### Server (`src/index.ts`)
Uses `Bun.serve()` with HTML imports:
- Routes defined in `routes` object
- HMR enabled in development via `development.hmr`

### Frontend Entry
- `src/index.html` → loads `src/frontend.tsx`
- `src/frontend.tsx` → renders `src/App.tsx`
- `src/App.tsx` → main app with all components

### API Layer (`src/lib/api.ts`)
- Ky HTTP client configured for SWAPI
- TypeScript interfaces for all resource types (Person, Planet, Film, etc.)
- Functions: `fetchResource`, `fetchByUrl`, `searchResource`, `searchAllResources`

### State Management (`src/store/swapi.ts`)
Zustand store with persist middleware:
- `currentResource` - active tab (people, planets, films, etc.)
- `page`, `itemsPerPage` - pagination state
- `globalSearchQuery/Results` - search state
- `cache`, `apiPageCache` - localStorage cached API responses
- Handles multi-page fetching (SWAPI returns 10 per page, app shows 24)

### URL State Sync (`src/App.tsx`)
Query parameters sync with app state:
- `?type=planets` - resource type
- `?page=2` - pagination
- `?q=luke` - search query
- `?detail=people/1` - detail view

### Styling
- Tailwind 4 with shadcn/ui (new-york style, dark theme only)
- `styles/globals.css` → theme variables
- `src/index.css` → animated star background

### UI Components (`src/components/ui/`)
shadcn/ui components: button, card, input, select, tabs, badge, skeleton, dialog

## Key Patterns

### Stale Response Detection
Async fetches capture current state, check if state changed before updating:
```ts
const fetchingResource = currentResource;
// ... await fetch ...
if (get().currentResource !== fetchingResource) return; // abort if stale
```

### Expandable Badges
Badges show related entity counts. Clicking expands to show names (fetched and cached). Clicking names opens detail modal.

### Detail View
Single entity view with full expandable badges. Accessed via modal "view full" link or direct URL.

## Bun Preferences

- `Bun.serve()` for HTTP (not Express)
- `Bun.file` for file I/O (not node:fs)
- Auto-loads `.env` files (no dotenv needed)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/travispulley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
