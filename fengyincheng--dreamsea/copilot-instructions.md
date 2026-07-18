## dreamsea

> This is a Node.js 22+ ES module application. `client/src/` contains the Phaser/Vite frontend: scenes live in `scenes/`, DOM controllers in `ui/`, and API access in `network/`. `server/` contains Express code split into `domain/`, `services/`, `persistence/`, `memory/`, and `church/`; keep `server/index.js` as the composition root and `server/app.js` injectable for tests. Shared contracts belong in `shared/`. Tests are in `test/*.test.js`, with Playwright specs in `test/browser/`. Runtime assets go in `client/public/`; editable artwork belongs in `ArtAssets/`. Scripts and prompt templates live in `scripts/` and `templates/`.

# Repository Guidelines

## Project Structure & Module Organization

This is a Node.js 22+ ES module application. `client/src/` contains the Phaser/Vite frontend: scenes live in `scenes/`, DOM controllers in `ui/`, and API access in `network/`. `server/` contains Express code split into `domain/`, `services/`, `persistence/`, `memory/`, and `church/`; keep `server/index.js` as the composition root and `server/app.js` injectable for tests. Shared contracts belong in `shared/`. Tests are in `test/*.test.js`, with Playwright specs in `test/browser/`. Runtime assets go in `client/public/`; editable artwork belongs in `ArtAssets/`. Scripts and prompt templates live in `scripts/` and `templates/`.

## Build, Test, and Development Commands

- `npm ci`: install the locked dependency set.
- `npm run dev:server`: run Express with file watching on port 3000.
- `npm run dev:client`: run Vite on port 5173 and proxy API requests.
- `npm test`: run Node unit and API tests serially.
- `npm run test:browser`: run Chromium Playwright tests against port 3100.
- `npm run build`: produce the frontend bundle in `client/dist/`.
- `npm run check`: run unit tests, build, and browser tests; use before opening a PR.

Copy `.env.example` to `.env` before local development.

## Coding Style & Naming Conventions

Follow the existing JavaScript style: two-space indentation, double quotes, semicolons, trailing commas in multiline literals, and explicit `.js` import extensions. Use `camelCase` for functions and variables, `PascalCase` for classes, and kebab-case filenames such as `home-state-service.js`. Keep domain logic pure where possible and inject external dependencies. No formatter or linter is configured, so match surrounding code.

## Testing Guidelines

Use `node:test` with `node:assert/strict`; name files `*.test.js` and describe behavior in test names. Browser flows use `*.spec.js`. Add regression coverage with every behavior change, preferring isolated temporary data directories and injected fakes. Run one test with `node --test test/window-scene.test.js`.

## Commit & Pull Request Guidelines

Recent history follows Conventional Commit prefixes, chiefly `feat:` and `fix:` (for example, `fix: retry explicit image gateway failures`). Write short, imperative subjects and keep each commit coherent. PRs should explain user-visible behavior, list verification commands, link relevant issues, and include screenshots or recordings for UI or asset changes. Call out configuration, persistence, or migration effects explicitly.

## Security & Configuration

Never commit `.env`, API keys, session secrets, provider credentials, or generated data from `server/data/`. Preserve server-side handling of coordinates and secrets. When adding configuration, document it in `.env.example` and avoid logging sensitive request data.

---
> Source: [fengyincheng/DreamSea](https://github.com/fengyincheng/DreamSea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
