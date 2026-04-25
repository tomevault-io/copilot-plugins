## mywebsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server at localhost:4321
npm run build     # Build production site to ./dist/
npm run preview   # Preview production build locally
```

## Architecture

This is an **Astro 5 static site** — a personal portfolio for Bruce Tseng (曾冠瑜), a computational linguistics graduate student at NTU.

**Framework**: Astro 5 with `@astrojs/react` integration for interactive components. Plain `.astro` components for pages; React (`.tsx`) for stateful interactive tools.

### Routing

File-based routing under `src/pages/`. Main sections:

- `/` → `src/pages/index.astro` — homepage grid showing latest from all collections
- `/about` → bilingual (CN/EN) profile page with a language toggle button
- `/blog/[id]`, `/research/[id]`, `/reading/[id]`, `/music/[id]` — dynamic routes using `getStaticPaths()`
- `/minesweeper` → `src/pages/minesweeper.astro` — Minesweeper game (React, `client:load`)
- `/hyperbolic` → `src/pages/hyperbolic.astro` — Poincaré disk visualization (React, `client:load`)
- `/split` → `src/pages/split.astro` — Trip expense splitter (React, `client:load`)

### Content Collections

All content lives in `src/content/` as Markdown files with YAML frontmatter. Schemas defined in `src/content/config.ts` using Zod. Fetch with `getCollection('blog')` etc. from `astro:content`.

Each collection has its own frontmatter schema:
- **blog**: mood, category, location, tech stack
- **research**: status, field, abstract, PDF links
- **reading**: book ratings
- **music**: artist, song name, YouTube links, lyrics
- **projects**: demoUrl, githubUrl (use `z.string()` not `z.string().url()` to allow relative paths)

### Layout & Styling

- `src/layouts/Layout.astro` — global wrapper with `<ClientRouter />` for page transitions, navbar, CSS variables, and click particle effect
- `src/components/navbar.astro` — sticky nav with live clock and language toggle (visible only on `/about`)
- Styles use scoped `<style>` blocks in each `.astro` file; global styles use `is:global`
- CSS variables for theming: `--ink`, `--paper`, `--accent`, `--muted`, `--border`
- Font: Noto Serif TC (serif), JetBrains Mono for code/game UI
- Paper/ink warm aesthetic; breakpoints at 768px and 850px
- React component pages use `<style is:global>` with namespaced CSS classes (e.g. `.ms-*`, `.hb-*`, `.sp-*`)

### Interactivity Pattern

- Plain `.astro` pages: client-side logic via inline `<script>` tags
- Scripts needing re-run after page transitions must listen for `astro:page-load` (not `DOMContentLoaded`)
- Interactive tools: React components in `src/components/<name>/`, mounted with `client:load` in the page

### Language Support

The `/about` page supports bilingual (Traditional Chinese / English) content via `data-lang` attributes on elements. The language toggle in the navbar switches a `data-lang` attribute on `<body>`, and CSS hides/shows the corresponding content.

### Interactive Projects

#### Minesweeper (`src/components/minesweeper/`)
- `useGameState.ts` — all game logic
- `MinesweeperGame.tsx` (root), `Board.tsx`, `Cell.tsx`, `GameHeader.tsx`, `DifficultyPanel.tsx`
- Styles: inline `<style>` inside `MinesweeperGame.tsx`

#### Poincaré Disk Visualization (`src/components/hyperbolic/`)
- `hyperbolicMath.ts` — Möbius add, distance, embedTree, geodesicArcPath
- `useHyperbolicState.ts`, `PoincareDisk.tsx` (root), `DiskCanvas.tsx`, `GeodesicEdge.tsx`, `NodeCircle.tsx`, `InfoPanel.tsx`
- 33 WordNet nodes depth 0–6; diskR=320 desktop / 180 mobile

#### Trip Expense Splitter (`src/components/split/`)
- `types.ts` — Member/Expense/Transaction/Step/MEMBER_COLORS
- `useSplitState.ts`, `SplitApp.tsx` (root), `StepIndicator.tsx`, `MembersStep.tsx`, `ExpenseForm.tsx`, `ExpensesStep.tsx`, `SettlementStep.tsx`
- localStorage key: `split-state-v1`
- Settlement: greedy max-creditor/max-debtor matching for minimum transactions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bbrruu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
