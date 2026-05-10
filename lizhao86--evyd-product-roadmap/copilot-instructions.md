## evyd-product-roadmap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EVYD Product Roadmap — a visual roadmap whiteboard for product planning. Vanilla JS frontend + Vercel Serverless API. Data persists in Vercel Blob (private access, signed URLs). UI is in Chinese.

## Running

```bash
npm install            # install @vercel/blob dependency
npx vercel dev         # local dev with API routes + Vercel Blob
# or pure static preview (no remote storage):
open index.html
```

Deployed at https://evyd-product-roadmap.vercel.app/ via Vercel (auto-deploys from GitHub).
Requires `BLOB_READ_WRITE_TOKEN` environment variable in Vercel project settings.

## Architecture

Main files in the root:

- **`index.html`** — static markup: toolbar, filter bar, roadmap body, modal form, tooltip
- **`app.js`** — all logic in one file (~1790 lines), organized into clearly commented sections:
  - Constants (fiscal year math, color palettes, layout dimensions)
  - State (global mutable state: `appData`, view/filter state, drag state)
  - Persistence (async `loadData()`/`saveData()` via `/api/data` endpoint)
  - Data helpers (module/pillar/project ordering, color assignment, row packing, filtering)
  - Render pipeline (`render()` → `renderValueView()` or `renderProjectView()` → `renderInnerGroup()` → `createBar()`)
  - Drag interactions (move bars, resize bars, reorder outer groups)
  - Tooltip, Modal, CSV import/export
  - Init (event binding)
- **`styles.css`** — all styles, CSS custom properties in `:root`
- **`api/data.js`** — Vercel Serverless Function: GET/PUT endpoints for Vercel Blob storage (private access, reads via `getDownloadUrl` signed URLs) with optimistic locking
- **`package.json`** — declares `@vercel/blob` dependency
- **`vercel.json`** — route rewrites + no-cache headers for API

## Key Concepts

- **Fiscal year**: April (month 4) through March. `toFiscalCol(calMonth)` converts calendar month 1–12 to fiscal column 0–11. All bar positioning uses fiscal columns.
- **Two views**: "Value view" groups by Pillar → Project. "Project view" groups by Project → Module. Both share `renderInnerGroup()` for the inner level.
- **Bar colors**: Assigned by module name via `MODULE_COLORS` palette (8 colors, cycled). Colors are session-stable, rebuilt on each `render()` via `rebuildColorMap()`.
- **Data model**: `appData.items[]` — each item has: `id`, `module`, `pillar`, `project`, `title`, `problem`, `description`, `outcome`, `author`, `collaborators[]`, `devType`, `pillarValue`, `startMonth` (1–12), `duration` (months). Order arrays: `moduleOrder`, `pillarOrder`, `projectOrder`. Top-level `version` (integer) and `lastModified` (ISO string) used for optimistic locking.
- **Remote storage**: Data stored in Vercel Blob (private access) via `api/data.js`. Blob reads use `getDownloadUrl()` for time-limited signed URLs. `loadData()` and `saveData()` are both `async`. All callers use `await`. On 409 Conflict (version mismatch), user is prompted to refresh.
- **Row packing**: `packRows()` uses greedy interval scheduling to stack overlapping bars vertically within a group.
- **Drag**: Three separate drag systems — bar move (`startDragMove`), bar resize (`startDragResize`), outer group reorder (`startPillarDrag`/`startProjectDrag`/`startModuleDrag`). All use mousedown/mousemove/mouseup on `document`.
- **CSV format**: 13 columns with both Chinese and English field names. Export uses UTF-8 BOM for Excel compatibility. Import auto-detects UTF-16 LE / UTF-8 BOM / UTF-8.
- **Dev types**: Three values — `New Contract`, `R&D`, `Maintenance` — each with abbreviation and color in `DEV_TYPE_CONFIG`.

## Conventions

- No framework, no TypeScript — keep it vanilla JS. Only npm dependency is `@vercel/blob` for the serverless function
- All DOM manipulation is imperative; `render()` rebuilds the entire roadmap body on every state change
- HTML escaping via `esc()` function — use it for all user-supplied strings inserted into innerHTML
- `saveData()` is async — increments `appData.version`, PUTs to `/api/data`, handles 409 conflict. All callers must `await` it
- localStorage is only used for UI preferences: `COLLAPSED_KEY` (fold state) and `LABEL_WIDTH_KEY` (column width)
- Cache-busting: `index.html` loads `app.js?v=16` — bump the version query param when making changes
- CSS variable `--label-width` controls the left label column width (user-resizable, persisted)

---
> Source: [lizhao86/EVYD-Product-Roadmap](https://github.com/lizhao86/EVYD-Product-Roadmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
