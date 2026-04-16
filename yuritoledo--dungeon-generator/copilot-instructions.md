## dungeon-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Shadowdark Tomb Generator — a procedural dungeon generator web app for the Shadowdark RPG. Generates randomized tomb dungeons with encounters, monsters, traps, and loot. Bilingual (en-US / pt-BR), deployed as a static site to GitHub Pages.

## Commands

```bash
pnpm dev              # Dev server with hot reload
pnpm build            # Production build to /build
pnpm preview          # Preview production build
pnpm check            # TypeScript + Svelte type checking
pnpm lint             # ESLint
pnpm lint:fix         # ESLint auto-fix
```

Package manager is **pnpm**. No test framework is configured.

## Tech Stack

- **Svelte 5** (runes: `$state`, `$effect`, `$bindable`) + **SvelteKit 2** (static adapter)
- **TypeScript** (strict mode) + **Vite 6**
- **HTML5 Canvas** for interactive dungeon map rendering
- Deployed to GitHub Pages at `/dungeon-generator` base path

## Architecture

### Data Flow

User selects tier/room count/density → `generateDungeon()` produces a `Dungeon` object → `buildLayout()` places rooms on a spatial grid → Canvas renders the map, Svelte components render room detail cards.

### Key Modules

- **`src/lib/generator.ts`** — Core generation algorithm: RNG helpers, monster/trap/loot assignment, room layout with grid-based placement. Entry point is `generateDungeon()`.
- **`src/lib/mapDraw.ts`** — Canvas drawing (rooms, corridors, secret passages, icons) and hit-testing for click-to-select.
- **`src/lib/types.ts`** — All TypeScript interfaces (`Room`, `Dungeon`, `Monster`, `Trap`, `Loot`, etc.).
- **`src/lib/data/`** — Static data tables organized by language key (`'en' | 'pt'`):
  - `encounters.ts` — Monsters (tiered 1-3), traps, loot (mundane/magic/rare)
  - `rooms.ts` — Room types, shapes with dimensions
  - `tomb.ts` — Tomb names, plot hooks, atmospheric features
  - `i18n.ts` — UI string translations (partially implemented)

### Components

- **`+page.svelte`** — Main page: controls, state management (all state lives here via Svelte runes), orchestrates generation and display.
- **`DungeonMap.svelte`** — Canvas wrapper, handles click events to select rooms.
- **`RoomCard.svelte`** — Displays a single room's details (monsters, traps, loot, connections).
- **`LangToggle.svelte`** — Language switcher (currently commented out; app hardcoded to pt-BR).

### i18n Status

Language toggle is commented out in `+page.svelte`. All data tables support both `'en'` and `'pt'` keys, but the UI currently hardcodes `'pt'` as the active language.

## Conventions

- State management uses Svelte 5 runes only — no external stores.
- All data tables are keyed by `'en' | 'pt'` language codes.
- The SvelteKit base path is `/dungeon-generator` (configured in `svelte.config.js`) for GitHub Pages deployment.
- CSS uses custom properties for theming (dark tomb aesthetic with Cinzel/IM Fell English fonts).
- GitHub Actions CI uses pnpm v9 + Node 22, deploys on push to main.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuritoledo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
