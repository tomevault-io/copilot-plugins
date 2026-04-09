## ryanbarclay-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (http://localhost:5174)
npm run build      # TypeScript check + Vite build
npm run tsc        # TypeScript type-check only (no emit)
npm run preview    # Preview production build locally
npm run deploy     # Build + deploy to GitHub Pages via gh-pages
```

No test suite is configured. TypeScript (`npm run tsc`) is the primary correctness check.

## Architecture Overview

This is a React 19 + TypeScript portfolio site deployed to GitHub Pages. Built with Vite, MUI v7, and React Router v6 using **HashRouter** (required for GitHub Pages compatibility).

### Routing

Two-level route config:
- `src/config/routes.tsx` — top-level nav routes (Home, Projects, About)
- `src/config/projectRoutes.tsx` — maps project IDs to page components under `/projects/:id`

`src/components/layout/MainContent.tsx` consumes both configs to render routes.

### Adding a New Project

1. Add an entry to `projectsData` in `src/data/projects.ts` with a unique `id`
2. Create the project page under `src/pages/projects/<id>/`
3. Register it in `src/config/projectRoutes.tsx` (maps `id` → component)

Project metadata (title, description, tags, links, `featured` flag) lives entirely in `projects.ts`. The `featured` flag controls visibility on the home page grid.

### Theme / Styling

- MUI theme defined in `src/theme/theme.tsx` with `darkTheme` / `lightTheme` variants
- Theme toggled via `ThemeContext` (`src/contexts/DarkModeContext.tsx`)
- `AppProvider` (`src/contexts/AppProvider.tsx`) wraps all contexts

### Terrain Generator Project (`src/pages/projects/terrain-generator/`)

The most complex project — a WebGL 3D terrain renderer using React Three Fiber. Key data flow:

1. `TerrainContext.tsx` holds all config state and exposes `generateTerrain()`
2. `useTerrainGen.ts` runs noise → heightmap → geometry pipeline on demand
3. `useLODSystem.ts` manages quadtree spatial partitioning for distance-based LOD
4. `TerrainCanvas.tsx` sets up the R3F `<Canvas>` with `TerrainMesh` and `WaterPlane`

**Important**: Terrain only regenerates when the user clicks "Generate Terrain" — size changes are not live because 512×512 vertex recalculation is expensive.

When adding a new control parameter: add to `TerrainConfig` in `types.ts` → initialize in `TerrainContext.tsx` → wire up in `useTerrainGen.ts` → add control component → import in `ControlPanel.tsx`.

### Deployment

The site deploys to `RyanBarclay.github.io` via `npm run deploy` (uses `gh-pages` to push `dist/` to the `gh-pages` branch). The `CNAME` file is copied into `dist/` during deploy to preserve the custom domain.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RyanBarclay)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RyanBarclay)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
