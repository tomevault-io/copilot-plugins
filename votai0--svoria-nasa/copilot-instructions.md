## svoria-nasa

> - Supports NASA Space Apps Exoplanet Hunter via an interactive web viewer; AI/ML runs outside this repo.

# Repository Guidelines

## Mission Focus & Data Inputs
- Supports NASA Space Apps Exoplanet Hunter via an interactive web viewer; AI/ML runs outside this repo.
- Source curated tables from NASA Exoplanet Archive or TESS; keep lightweight transforms in `src/services/data`.
- Store presentation-ready CSV/JSON under `public/data` and document provenance and filters.

## Project Structure & Module Organization
- `src/main.tsx` mounts React via Vite; `App.tsx` drives the `@react-three/fiber` solar-system viewer and UI panels.
- Scene widgets live in `src/components`; shared enums and interfaces belong in `src/types` and `src/constants`.
- Data adapters, ephemeris helpers, and fetchers live in `src/services`; pure utilities stay in `src/utils`.
- Optimized textures sit in `src/assets`; large mission files load from `public`.

## Build, Test, and Development Commands
- `npm run dev` — start the Vite dev server with hot reload at `http://localhost:5173`.
- `npm run build` — run `tsc -b` then emit the production bundle to `dist/`.
- `npm run preview` — serve the last build for demo reviews.
- `npm run lint` — enforce the shared ESLint profile with React Hooks and TypeScript checks.

## Coding Style & Naming Conventions
- Use TypeScript; declare prop and service contracts in `src/types` for reuse.
- Follow two-space indentation, single quotes, trailing commas, and comment only when orbital math or data transforms are opaque.
- Components and hooks use `PascalCase`; utilities and constants prefer `camelCase` or `SCREAMING_SNAKE_CASE` when representing mission flags.

## Testing Guidelines
- Introduce `vitest` and `@testing-library/react`; mock WebGL contexts and time utilities to keep tests deterministic.
- Validate new targets with orbital snapshots and list manual QA steps (camera reset, URL share, dataset switch) in PR notes.
- Track mission journeys (target search, sector highlight, share link) in tests to catch regressions.

## Commit & Pull Request Guidelines
- Match history: short, lower-case, present-tense subjects (e.g., `search bar eklendi`); keep each commit to one deliverable.
- Reference Space Apps issue IDs or challenge milestones, and flag texture/data updates explicitly.
- Provide concise PR descriptions with gifs for interactive features and manual checks for target selection, time controls, and share links.
- Request review from scene or data stewards when touching orbital physics, shader setup, or shared constants.

## Scene & Asset Notes
- Keep textures under ~5 MB, compress NASA imagery, and document licensing.
- When adding exoplanet presets, update `src/constants/exoplanets.ts`, sync narrative copy in UI panels, and ensure `parseURLParams` handles the new identifier gracefully.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Votai0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Votai0)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
