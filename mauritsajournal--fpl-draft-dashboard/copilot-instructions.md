## fpl-draft-dashboard

> Auto-updating dashboard for FPL Draft League 820, showing standings, H2H comparisons, player stats, gameweek trends, point predictions, and more. Hosted free on GitHub Pages.

# FPL Draft Dashboard — FC Driegangendiner

## Project Overview
Auto-updating dashboard for FPL Draft League 820, showing standings, H2H comparisons, player stats, gameweek trends, point predictions, and more. Hosted free on GitHub Pages.

## Tech Stack
- **Framework:** Astro 5.x (static site generator)
- **Interactive components:** Svelte 5.x islands
- **Styling:** Tailwind CSS 4.x (via `@tailwindcss/vite`)
- **Charting:** Chart.js 4.x (tree-shaken)
- **Language:** TypeScript (strict mode)
- **Data pipeline:** Node.js 22 scripts run via GitHub Actions cron
- **Hosting:** GitHub Pages (free)
- **Testing:** Vitest

## Key Identifiers
- **League ID:** 820
- **League name:** FC Driegangendiner
- **Season:** 2025-26
- **Maurits Koot entry_id:** 27628 (highlight in UI)
- **8 managers** with league_entry IDs in `data/league.json`

## Folder Structure
```
src/              — Astro pages, components, layouts, styles
scripts/          — Data fetch and transform scripts (TypeScript, run via tsx)
scripts/types/    — TypeScript types for API responses and dashboard data
scripts/lib/      — Shared utilities (fetch, player resolver)
data/             — JSON data files (committed to repo, updated by Actions)
data/picks/       — Per-GW picks (gw01.json - gw38.json)
data/live/        — Per-GW live stats
tests/            — Vitest test files
docs/             — Architecture docs, tickets, research
.github/workflows/ — CI/CD workflows
```

## Data Flow
1. GitHub Actions cron (4x daily) runs `scripts/fetch-all.ts`
2. Fetches 8 API endpoints, writes JSON to `data/`
3. Runs `scripts/transform.ts` to produce `data/dashboard.json`
4. Commits data changes, builds Astro site, deploys to GitHub Pages
5. Astro reads `dashboard.json` at build time (no client-side API calls)

## How to Run Locally
```bash
npm install
npm run fetch        # Fetch latest data from FPL API
npm run transform    # Build dashboard.json from raw data
npm run dev          # Start Astro dev server
npm run build        # Build static site to dist/
npm run test         # Run Vitest tests
```

## Conventions
- Astro components (`.astro`) for static content, Svelte islands for interactivity
- Data imported at build time: `const mod = await import('../../data/dashboard.json')`
- All data pre-computed in transform step; frontend is rendering only
- Commit format: `feat(#T-XXX): description` or `fix(#T-XXX): description`
- Code comments in English; Dutch league/player names in data

## FPL Draft API Endpoints (no auth needed)
- Bootstrap: `GET https://draft.premierleague.com/api/bootstrap-static`
- League: `GET https://draft.premierleague.com/api/league/820/details`
- Element status: `GET https://draft.premierleague.com/api/league/820/element-status`
- Entry event: `GET https://draft.premierleague.com/api/entry/{id}/event/{gw}`
- Live event: `GET https://draft.premierleague.com/api/event/{gw}/live`
- Transactions: `GET https://draft.premierleague.com/api/draft/league/820/transactions`
- Draft choices: `GET https://draft.premierleague.com/api/draft/820/choices`
- Fixtures: `GET https://fantasy.premierleague.com/api/fixtures/`

## ESPN Premier League API (no auth needed)
Used for the "Draft x PL" page that combines real PL data with FPL draft data.
- Teams: `GET https://site.api.espn.com/apis/site/v2/sports/soccer/eng.1/teams`
- Team roster: `GET https://site.api.espn.com/apis/site/v2/sports/soccer/eng.1/teams/{espnTeamId}/roster`
- Team stats: `GET https://sports.core.api.espn.com/v2/sports/soccer/leagues/eng.1/seasons/2025/types/1/teams/{espnTeamId}/statistics`
- Player stats: `GET https://sports.core.api.espn.com/v2/sports/soccer/leagues/eng.1/seasons/2025/types/1/teams/{espnTeamId}/athletes/{athleteId}/statistics`
- Standings: `GET https://site.web.api.espn.com/apis/v2/sports/soccer/eng.1/standings?season=2025`
- Data cached in `data/espn/` (team-stats.json, player-stats.json, rosters.json, standings.json, player-mapping.json)
- Fetch script: `scripts/fetch-espn.ts` (run via `npm run fetch:espn`)
- Transform: `scripts/lib/espn-transform.ts` (produces `draftXPL` section in dashboard.json)
- FPL team ID -> ESPN team ID mapping in fetch-espn.ts `FPL_TO_ESPN` constant
- Player matching uses fuzzy name matching (last name, normalized accents, hyphenated names)
- ESPN stat fields use `name` property (camelCase, e.g., `totalGoals`, `shotsOnTarget`, `foulsCommitted`)

## Important API Notes
- Draft API `events` is `{ current, next, data: [...] }` (NOT a flat array)
- Transactions response is `{ transactions: [...] }` (NOT a bare array)
- Player xG/xA fields are strings ("6.84"), not numbers
- `element_type`: 1=GK, 2=DEF, 3=MID, 4=FWD
- `matches_played` in standings = total scheduled (38), not actually played

## GitHub Actions
- `fetch-and-deploy.yml`: cron + manual; fetches data + builds + deploys (single workflow to avoid GITHUB_TOKEN limitation)
- `deploy.yml`: code-push triggered; builds + deploys
- `keepalive.yml`: weekly Monday cron to prevent disable after 60 days

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mauritsajournal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
