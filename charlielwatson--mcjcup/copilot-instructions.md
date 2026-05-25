## mcjcup

> Watson-Southey Golf Classic — a golf tournament website for three players (Max Watson, Charlie Watson, Jacob Southey) competing across three New Zealand venues in match play format. Built with TanStack Start and deployed on Netlify.

# AGENTS.md

## Project Overview

Watson-Southey Golf Classic — a golf tournament website for three players (Max Watson, Charlie Watson, Jacob Southey) competing across three New Zealand venues in match play format. Built with TanStack Start and deployed on Netlify.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | TanStack Start |
| Frontend | React 19, TanStack Router v1 |
| Build | Vite 7 |
| Styling | Tailwind CSS 4 |
| Language | TypeScript 5.7 (strict mode) |
| Deployment | Netlify |

## Directory Structure

```
public/
  tournament-logo.png   # 2026 tournament banner (source of truth for branding)
  favicon.ico
src/
  routes/
    __root.tsx          # Root shell: sets page title, imports styles.css
    index.tsx           # Single-page app: schedule, leaderboard, past winners, rules
  router.tsx            # TanStack Router setup
  styles.css            # Global Tailwind + NZ bush color palette via CSS variables
netlify.toml            # Build config: vite build → dist/client, dev port 8888
vite.config.ts          # Vite plugins: TanStack Start, Netlify, Tailwind
tsconfig.json           # Strict mode, @/* alias for src/*
```

## Color Palette (NZ Bush Theme)

All colors defined as CSS custom properties in `styles.css` and registered via `@theme` for Tailwind utility access (`bg-[--color-bush-dark]` syntax):

| Variable | Hex | Usage |
|---|---|---|
| `--color-bush-darkest` | #0f2318 | Header bg, body text |
| `--color-bush-dark` | #1a3d2b | Section headers, nav |
| `--color-bush-mid` | #2d5a3d | Nav bar bg |
| `--color-bush-fern` | #4a7c59 | Secondary text, accents |
| `--color-cream` | #f2ede3 | Page background |
| `--color-gold` | #c9a84c | Trophy, position badges |
| `--color-parchment` | #e8dfcc | Card/table backgrounds |

## Key Concepts

### File-Based Routing (TanStack Router)

Routes are defined by files in `src/routes/`:
- `__root.tsx` — Root layout wrapping all pages
- `index.tsx` — Route for `/` (all sections: schedule, leaderboard, past winners, rules)
- `api.*.ts` — Server API endpoints

### Data Architecture

All data (schedule, leaderboard, past winners, rules) is **static** inside `src/routes/index.tsx`. This is intentional — no backend needed for initial build. When live scores need tracking, add Netlify Database (Drizzle ORM via `@netlify/database`).

## Development Commands

```bash
npm run dev      # Start dev server on port 3000
npm run build    # Production build
```

## Conventions

### Naming
- Components: PascalCase
- Routes: kebab-case files

### Styling
- Tailwind CSS 4 utility classes
- CSS variables for theme tokens: `bg-[--color-variable-name]`
- No component library — all UI is hand-crafted

### TypeScript
- Strict mode enabled
- Import paths use `@/` alias for `src/*`
- Type-only imports with `type` keyword

## Non-Obvious Decisions

- All content lives in a single route (`index.tsx`) rather than separate pages — the site is a single-scroll experience with anchor link navigation.
- `public/tournament-logo.png` was uploaded by the user and is the primary brand asset. Do not delete or rename it.
- Leaderboard tab state is client-only React state — no server data fetching until real scores are needed.

---
> Source: [charlielwatson/mcjcup](https://github.com/charlielwatson/mcjcup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
