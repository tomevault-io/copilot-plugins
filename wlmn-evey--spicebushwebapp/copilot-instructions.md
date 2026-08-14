## spicebushwebapp

> - `app/`: Astro site, with pages in `app/src/pages/`, components in `app/src/components/`, shared utilities under `app/src/lib/`, and tests in `app/tests/` or near code.

# Repository Guidelines

## Project Structure & Module Organization
- `app/`: Astro site, with pages in `app/src/pages/`, components in `app/src/components/`, shared utilities under `app/src/lib/`, and tests in `app/tests/` or near code.
- `scripts/`: Operational helpers for deployment, migration, and maintenance tasks.
- `src/`: Shared root-level services used by non-app tooling.
- `docs/`, `journal/`, `debug/`: Internal documentation and operational notes; review before touching infrastructure or incident playbooks.

## Build, Test, and Development Commands
- `npm run dev`: Launches Netlify dev server with functions/proxy (run from repo root).
- `cd app && npm run dev`: Starts the Astro app server for faster front-end iteration.
- `cd app && npm run build`: Produces `app/dist/` for deployment and staging previews.
- `cd app && npm run lint && npm run format:check`: Verifies linting and Prettier formatting.
- `cd app && npm run test` / `npm run test:coverage`: Executes Vitest suites with optional coverage report.
- `cd app && npm run test:e2e [-- --ui]`: Runs Playwright E2E tests; add `--ui` for interactive runner.

## Coding Style & Naming Conventions
- Use TypeScript in `app/` with 2-space indentation; follow ESLint (`app/.eslintrc.json`) and Prettier defaults.
- Components are PascalCase (`Button.tsx`); pages are kebab-case route files (`tuition-policy.astro`).
- Utilities follow camelCase naming in `app/src/lib/` or `app/src/utils/`.

## Testing Guidelines
- Write unit/integration tests with Vitest (`*.test.ts` or `*.spec.ts`) close to source or under `app/src/test/`.
- Place Playwright scenarios in `app/e2e/` or `app/tests/`; keep fixtures minimal and deterministic.
- Run `npm run test` and `npm run test:e2e` locally before pushing; aim to cover new logic and routes.

## Commit & Pull Request Guidelines
- Use conventional prefixes (e.g., `feat(auth):`, `fix(netlify):`, `[BUILD-FIX]`) observed in history.
- PRs should describe scope, link issues, list key commands run, and include screenshots or Netlify preview URLs for UI changes.

## Security & Configuration Tips
- Do not commit secrets; populate `app/.env.local` from `app/.env.example` and rely on Netlify env vars.
- Run `cd app && npm run setup:dev` to validate required variables before development.
- Keep API endpoints configurable via environment variables.

---
> Source: [wlmn-evey/SpicebushWebapp](https://github.com/wlmn-evey/SpicebushWebapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
