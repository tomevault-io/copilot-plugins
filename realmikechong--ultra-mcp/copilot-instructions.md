## ultra-mcp

> - `src/`: TypeScript sources for the CLI and MCP server.

# Repository Guidelines

## Project Structure & Module Organization
- `src/`: TypeScript sources for the CLI and MCP server.
  - Key modules: `src/cli.ts`, `src/server.ts`, `src/commands/`, `src/providers/`, `src/config/`, `src/db/`, `src/api/`, `src/utils/`, `src/vector/`.
- `src/__tests__/`: Unit, integration, and e2e tests; shared `setup.ts`.
- `drizzle/`: SQL schema/migrations and artifacts.
- `web/`: Dashboard app (built separately); output in `dist-web/`.
- Build outputs: `dist/` (CLI), `dist-web/` (dashboard).

## Build, Test, and Development Commands
- Install deps: `bun install` (preferred) or `npm install`.
- Build CLI: `bun run build` → emits `dist/cli.js` (executable) and copies `drizzle/`.
- Build dashboard: `bun run build:dashboard` (runs inside `web/`).
- Build all: `bun run build:all` (CLI + dashboard).
- Dev watch (CLI): `bun run dev`.
- Run dashboard in dev via CLI: `bun run dev:api` (builds then serves dashboard with `--dev`).
- Tests: `bun run test` (default), `bun run test:all`, coverage `bun run test:coverage`, UI `bun run test:ui`, e2e `bun run test:e2e`.

## Coding Style & Naming Conventions
- Language: TypeScript (ES2022 modules). Indentation: 2 spaces.
- Filenames: kebab-case for files/dirs (e.g., `start-server.ts`); tests end with `.test.ts`.
- Linting: ESLint + `@typescript-eslint`. Run `bun run lint` or `bun run lint:fix`.
- Console usage: `no-console` enforced except in CLI, config, utils, API, tests, and mocks (see `eslint.config.js`).

## Testing Guidelines
- Framework: Vitest (Node environment). Setup file: `src/__tests__/setup.ts`.
- Locations: colocated `*.test.ts` and suites under `src/__tests__/`.
- Coverage: V8 provider; reporters text/json/html. Default excludes `e2e` and `web`; use `test:all` to include everything.

## Commit & Pull Request Guidelines
- Commit style: Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`) as used in history.
- PR requirements: clear summary, rationale, linked issues, screenshots/GIFs for `web/` changes, and test notes.
- Quality gates: green tests, no lint errors, update docs/tests for behavior changes.

## Security & Configuration Tips
- Never commit secrets or `.env` values; configure locally via `npx -y ultra-mcp config`.
- Drizzle artifacts live in `drizzle/`; verify migrations when touching schema.
- MCP debugging: `npx @modelcontextprotocol/inspector node dist/cli.js` after `bun run build`.

---
> Source: [RealMikeChong/ultra-mcp](https://github.com/RealMikeChong/ultra-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
