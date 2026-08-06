## panhub-shenzjd-com

> This repository is a Nuxt 4 + TypeScript app with server-side search aggregation logic.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Nuxt 4 + TypeScript app with server-side search aggregation logic.

- `pages/` and `app.vue`: UI routes and page-level Vue components.
- `composables/`: client-side reusable logic (for example `useSearch.ts`, `useSettings.ts`).
- `server/api/`: HTTP endpoints (`*.get.ts`, `*.post.ts`).
- `server/core/`: core services, plugins, cache, types, and utilities.
- `config/`: runtime plugin/channel configuration.
- `test/unit/`: Vitest unit tests (`*.test.ts`).
- `test/*.mjs`: API/integration-oriented test scripts.
- `public/` and `assets/`: static files and styling assets.
- `data/`: local runtime data (JSON hot-search persistence in local/Docker).

## Build, Test, and Development Commands
Use `npm` (lockfile is `package-lock.json`).

- `npm run dev`: start local development server.
- `npm run build`: build production bundle.
- `npm run preview`: preview the production build locally.
- `npm run generate`: generate static output when needed.
- `npm test`: run all unit tests with Vitest.
- `npm run test:watch`: run tests in watch mode.
- `npm run test:coverage`: generate text/json/html coverage reports.
- `npm run test:api`: run API test script (`test/api.test.mjs`).
- `npm run deploy:cf`: deploy to Cloudflare Workers via Wrangler.

## Coding Style & Naming Conventions
- Language: TypeScript (`.ts`) and Vue SFC (`.vue`), ESM modules.
- Indentation: 2 spaces; keep semicolons and double quotes consistent with existing files.
- Naming:
  - Vue composables: `useXxx.ts`.
  - Server routes: `name.get.ts` / `name.post.ts`.
  - Unit tests: `*.test.ts` under `test/unit/`.
- Keep changes focused; prefer small, testable functions in `server/core/services`.

## Testing Guidelines
- Framework: Vitest (`vitest.config.ts`), Node environment, globals enabled.
- Coverage provider: V8; reporters include `text`, `json`, and `html`.
- Add/adjust unit tests whenever behavior in `server/core/**` changes.
- Run `npm test` before opening a PR; use `npm run test:coverage` for critical logic updates.

## Commit & Pull Request Guidelines
- Follow Conventional Commits, as used in history: `feat:`, `fix:`, `refactor:`, `delete:`.
- Keep commit subjects short and imperative; one logical change per commit.
- PRs should include:
  - clear summary of behavior changes,
  - linked issue (if applicable),
  - test evidence (command + result),
  - UI screenshots/GIFs for page/component updates.

## Security & Configuration Tips
- Do not commit secrets or local runtime data from `data/`.
- Validate new external plugins/sources in `server/core/plugins/` with timeout and retry-safe patterns.

---
> Source: [wu529778790/panhub.shenzjd.com](https://github.com/wu529778790/panhub.shenzjd.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
