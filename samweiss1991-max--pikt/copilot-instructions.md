## pikt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo contains two projects:

1. **pickt-app/** — The main project. A Vite React SPA for a talent recruitment marketplace (pickt). This is the source of truth for all frontend and backend logic.
2. **Pikt v1.html** — A legacy browser-based table tennis game (HTML5 Canvas, single file).

## Running pickt-app

```bash
cd pickt-app
npm install
npm run dev       # Vite dev server on http://localhost:5173
```

## pickt-app Architecture

- **Framework**: React 19 + Vite 8 + React Router DOM 7
- **Styling**: Tailwind CSS 4 + custom CSS with design tokens (`src/styles/tokens.css`)
- **Backend**: Supabase (auth, database, storage, edge functions)
- **Design**: Light cream/forest green palette, Manrope font

### Key directories

- `src/pages/` — Route pages (Dashboard, Discovery, Marketplace, Shortlist, Refer, etc.)
- `src/components/` — Reusable components (layout, marketplace, shared, filters, unlock)
- `src/context/` — React contexts (Auth, Search, ViewMode)
- `src/lib/` — Utilities (supabase client, queries, ranking, sanitization, types)
- `src/data/` — Discovery options, mock data
- `src/styles/` — Global CSS, tokens, animations, responsive
- `supabase/functions/` — 3 Edge Functions (create-candidate, cv-url, unlock-candidate)
- `db/migrations/` — 18 Supabase migration SQL files
- `extension/` — Chrome extension for ATS integration
- `docs/` — Design specs (DESIGN_TOKENS.md, REDESIGN.md, prompt.txt)

### Data flow

- Most reads go directly to Supabase via `src/lib/supabaseQueries.js` (RLS-protected)
- Server-only operations (CV signed URLs, unlock + email, candidate creation) use Supabase Edge Functions called via `supabase.functions.invoke()`
- Seed data in `src/lib/candidatesSeed.ts` and `src/lib/seedData.js` for dev/demo mode

### Important patterns

- Never strip parallax, scroll reveal, hover-lift, or press-scale animations when rewriting pages
- Candidate discovery (ghost cards, tray, chips) lives in Marketplace, NOT Dashboard
- The app has 6 view modes: stack, carousel, matrix, tinder, compact, focus

## Git Workflow

All changes should be committed with clear messages and pushed to GitHub (origin: `samweiss1991-max/Pikt`). The repo uses the `master` branch.

---
> Source: [samweiss1991-max/Pikt](https://github.com/samweiss1991-max/Pikt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
