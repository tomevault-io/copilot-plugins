## plic-o-plic-team-res

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install       # Install dependencies
npm run dev       # Start dev server (Vite, hot reload)
npm run build     # Production build → ./dist
npm run preview   # Serve the built dist locally
```

There are no tests or linting configured.

## Architecture

Single-page React app (Vite + React 19) that calculates team standings from sports competition XLS/XLSX files. Deployed to GitHub Pages at `/plic-o-plic-team-res/` via `.github/workflows/deploy.yml`.

All source code is in `src/`:

- **`parseXls.js`** — pure data logic (no React). Exports three functions:
  - `parseXlsFile(file)` — reads a `File` object via `FileReader`, parses it with the `xlsx` library. Handles Windows-1251 encoding by manually re-decoding bytes. Expects XLS columns: `fam` (name), `grup` (age/gender group), `kom2` (team name), `m_1` (place), `o_1` (points), `r_1` (rank). Skips rows without `fam`, `grup`, or `kom2`.
  - `getUniqueGroups(participants)` — extracts unique group names and auto-detects gender from group name (`Ж` → female, `Ч` → male, otherwise `null`).
  - `computeTeamResults(participants, groupGenders)` — computes team standings separately for male/female groups. Per group per team, takes top 2 participants by `o_1` (points); sums those points into `totalPoints`. Returns `{ male: [...teams], female: [...teams] }` sorted by `totalPoints` descending. Each team has `name`, `totalPoints`, and `contributors` (all members with `used: true/false`).

- **`App.jsx`** — all UI components in one file:
  - `FileUpload` — drag-and-drop or click-to-upload zone
  - `GroupGenderSelector` — prompts user to classify groups with unknown gender
  - `GroupsList` — shows all groups with toggle buttons to switch gender assignment
  - `TeamCard` — collapsible card showing a team's rank, points, and per-group member breakdown
  - `ResultsSection` — renders a list of `TeamCard`s for one gender division
  - `App` (default export) — root component managing state: `participants`, `groups`, `groupGenders`, `fileName`, `error`

- **`App.module.css`** — CSS Modules scoped styles for all components above
- **`index.css`** — global reset/base styles
- **`main.jsx`** — React DOM entry point

## Key data flow

1. User uploads XLS → `parseXlsFile` → array of participant objects
2. `getUniqueGroups` identifies groups; gender auto-detected from group name or left for user to set
3. `computeTeamResults(participants, groupGenders)` re-runs reactively whenever `groupGenders` state changes
4. Results rendered as two side-by-side sections (male / female)

## XLS format expected

The input file must have these column headers (case-sensitive): `fam`, `grup`, `kom2`, `r_1`, `m_1`, `o_1`. Files are typically Windows-1251 encoded (Ukrainian/Cyrillic). The `codepage: 1252` option is passed to `xlsx` as a workaround, and cells are then manually re-decoded as Windows-1251.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VadymKolesnyk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
