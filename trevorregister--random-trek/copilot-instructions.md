## random-trek

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # dev server at http://localhost:3000
npm run build    # production build
npm run preview  # preview production build
```

No test suite or linter is configured.

## Architecture

Single-page Vue 3 app (no router, no Pinia/Vuex). All state lives in `src/App.vue` using `ref()`.

**Data flow:**
- 12 Star Trek series are imported as static JSON arrays from `src/data/*.json` (each episode: `{ season, number, title, synopsis, airDate }`)
- Each series is wrapped in a `ref()` — the refs themselves (not their `.value`) are stored in `shows` and `queryShows` arrays so identity comparison (`===`) can be used to match a show ref to its title/image
- `queryShows` is the active filter; clicking a `ShowCard` calls `handleUpdateFilter` which adds/removes the show ref from `queryShows`
- "Engage!" picks a random episode from `queryShows` via lodash shuffle
- Filter state persists to `localStorage` under key `random-trek-filter` as an array of show title strings

**Show identity pattern:** Because shows are tracked by ref identity, `getShowTitle(show)` and `getShowFromTitle(title)` exist as paired switch-statements to convert between a ref and its display name. Adding a new show requires updating both switches, the `shows`/`queryShows` arrays, and `getImage()`.

**UI components** (`src/components/ui/`): minimal shadcn-style primitives using `class-variance-authority`. `cn()` in `src/lib/utils.js` merges Tailwind classes. The `@` alias resolves to `src/`.

**Styling:** Tailwind CSS v4 (via `@tailwindcss/vite` plugin, no `tailwind.config.js`). Star Trek amber/gold color palette throughout.

---
> Source: [trevorregister/random-trek](https://github.com/trevorregister/random-trek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
