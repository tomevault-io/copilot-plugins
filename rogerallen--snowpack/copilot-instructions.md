## snowpack

> This file provides foundational context and operational rules for Gemini CLI within the Snowpack Tracker project.

# Gemini Context & Mandates

This file provides foundational context and operational rules for Gemini CLI within the Snowpack Tracker project.

## Project Overview

Snowpack Tracker is a full-stack dashboard for visualizing historical SNOTEL (Snow Telemetry) data, primarily focused on Oregon mountain stations. It enables users to compare current snow depth and SWE (Snow Water Equivalent) against historical seasons.

## Tech Stack

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS.
- **Data Fetching:** @tanstack/react-query (Caching: 5m stale, 30m GC).
- **Visualization:** Plotly.js (Seasonal charts), React-Leaflet (Station map).
- **Backend:** Node.js, Express, Pino (Logging).
- **API:** Hybrid model: Fetches historical POR from NRCS CSV and recent (365d) from Powderlines.
- **Data Processing:** Downsamples daily data to 3-day intervals (122 samples per season starting Sept 1).
- **Quality Logic:** 14-day zero-detection threshold (Oct-May) to flag sensor failures.
- **Database:** SQLite v4 with structured `snow_data` and `stations` tables.

## Core Mandates

### Operational Rules

- **No Line Numbers:** When providing copy-pasteable text, code blocks, or commit messages, **NEVER** include line numbers (e.g., `1 | ...`). Provide clean, raw text that can be used immediately.
- **Security:** Never log or commit the `.env` file contents.

### Engineering Standards

- **Data Flow:** Keep heavy data transformations (like seasonal grouping) on the backend to maintain a snappy frontend.
- **Strict Typing:** Always use the defined TypeScript interfaces in `src/hooks/useSnowData.ts` for snow data structures.
- **Testing:** Maintain high coverage for `server/services/snowService.js` as it contains the core business logic.

## Common Commands

- `npm run dev`: Starts both Vite and the Express server concurrently.
- `npm test`: Runs Vitest for both frontend and backend tests.
- `npm run build`: Production build and type check.
- `npm run lint`: Run lint checks on code. Use this after code changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rogerallen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rogerallen)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
