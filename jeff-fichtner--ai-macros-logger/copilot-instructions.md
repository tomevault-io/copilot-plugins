## ai-macros-logger

> Auto-generated from all feature plans. Last updated: 2026-02-20

# ai-macros-logger Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-20

## Active Technologies
- TypeScript 5.9, Node.js 20 LTS + Vitest 3.x, @testing-library/react, happy-dom (002-test-suite)
- N/A (test-only feature) (002-test-suite)
- TypeScript 5.9, Node.js 20 LTS + React 19, Vite 7, Zustand 5, Azure Functions v4, Tailwind CSS v4 (003-multi-provider-ai)
- Browser localStorage (Zustand persist), Google Sheets (data) (003-multi-provider-ai)
- Google Sheets (flat rows with 2 new columns: Group ID, Meal Label) (004-meal-grouping)
- TypeScript 5.x, Node.js 20 LTS + React 19, Vite, Zustand 5, Tailwind CSS v4 (005-delete-entries)
- Google Sheets API (single "Log" sheet per user) (005-delete-entries)
- TypeScript 5.9, Node.js 20 LTS + GitHub Actions, Azure Static Web Apps, `Azure/static-web-apps-deploy@v1` action, `jq` (CI only) (007-versioning-ci-cd)
- `version.json` at repo root (committed to git) (007-versioning-ci-cd)

- TypeScript 5.x, Node.js 20 LTS + React 19, Vite, React Router v7, Zustand, Tailwind CSS v4, Azure Functions v4 (Node.js model) (001-macro-nutrition-logger)

## Project Structure

```text
frontend/          # React + Vite SPA (Azure Static Web Apps)
  src/
    pages/         # FoodLog, Settings
    components/    # Layout, FoodInput, ParseResult, MacroSummary, EntryHistory, OnboardingGuide
    hooks/         # useSettings, useFoodLog, useGoogleAuth
    services/      # api, claude, sheets, oauth
    types/         # Shared TypeScript interfaces
api/               # Azure Functions v4 (managed functions)
  src/functions/   # parse, oauthToken, oauthRefresh
specs/             # Feature specifications (speckit)
```

## Commands

npm test && npm run lint

## Code Style

TypeScript 5.x, Node.js 20 LTS: Follow standard conventions

## Recent Changes
- 007-versioning-ci-cd: Added TypeScript 5.9, Node.js 20 LTS + GitHub Actions, Azure Static Web Apps, `Azure/static-web-apps-deploy@v1` action, `jq` (CI only)
- 006-refine-parse-results: Added TypeScript 5.x, Node.js 20 LTS + React 19, Vite, Zustand 5, Tailwind CSS v4
- 005-delete-entries: Added TypeScript 5.x, Node.js 20 LTS + React 19, Vite, Zustand 5, Tailwind CSS v4


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeff-fichtner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
