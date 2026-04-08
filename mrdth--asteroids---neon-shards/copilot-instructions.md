## asteroids-neon-shards

> The Phaser game source lives in `src/`, organised by feature (e.g., `core/` bootstraps the game, `scenes/` drives flow, `systems/` holds gameplay logic). Tests mirror the runtime folders under `tests/`, using matching filenames for unit, system, and integration coverage. Static assets ship from `assets/` while public HTML scaffolding sits in `public/`; Vite build presets reside in `vite/`, and generated bundles land in `dist/`.

# Repository Guidelines

## Project Structure & Module Organization

The Phaser game source lives in `src/`, organised by feature (e.g., `core/` bootstraps the game, `scenes/` drives flow, `systems/` holds gameplay logic). Tests mirror the runtime folders under `tests/`, using matching filenames for unit, system, and integration coverage. Static assets ship from `assets/` while public HTML scaffolding sits in `public/`; Vite build presets reside in `vite/`, and generated bundles land in `dist/`.

## Build, Test & Development Commands

- `npm install`: sync dependencies before your first run or when package updates land.
- `npm run dev`: start the Vite dev server with the logging overlay (`log.js`) for quick iteration.
- `npm run build`: produce a production bundle using `vite/config.prod.mjs`.
- `npm run test`: open Vitest in watch mode with the Phaser mocks from `tests/setup.ts`.
- `npm run test:run`: execute the full suite once, ideal for CI.
- `npm run test:ui`: launch the Vitest UI when you need to debug snapshots or flaky specs.

## Coding Style & Naming Conventions

Target TypeScript 5.x with ES modules and Phaser 3 APIs. Use two-space indentation, trailing semicolons, and double quotes (existing files are consistent). Prefer descriptive class names (`AsteroidSpawner`, `ShardManager`) and align helper files under `utils/`. Import project modules via the `@/` alias instead of relative paths when referencing `src/`.

## Testing Guidelines

Vitest with a `jsdom` environment is mandatory for new code; include focused unit tests plus higher-level integrations when behavior spans systems. Name files `*.test.ts` alongside the feature (`tests/systems/ShardManager.test.ts`) and reset spies between cases. Run `npm run test:run` before pushing, and keep Phaser mocks in `tests/setup.ts` up to date when engine contracts change.

## Commit & Pull Request Guidelines

Follow the conventional `type(scope): summary` pattern (`feat(gameplay): add shard attraction`); keep summaries under 72 characters and use scopes that map to repo folders. Commits should remain atomic: refactors separate from feature commits, tests updated with their code. Pull requests need a concise description of the change, screenshots or GIFs for visible updates, links to relevant issues, and a checklist of manual or automated tests you ran.

## Assets & Configuration Tips

Place art and audio in `assets/` and reference them via Phaser loaders; avoid committing generated textures. Update `README.md` and `docs/` when gameplay rules shift. Adjust renderer, physics, or logging defaults through `vite/config.dev.mjs` and `vite/config.prod.mjs` rather than ad-hoc environment variables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrdth)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mrdth)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
