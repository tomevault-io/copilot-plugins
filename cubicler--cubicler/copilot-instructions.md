## cubicler

> - Source: `src/` — core modules in `core/` (services), `transport/` (`agent/`, `mcp/`), `middleware/`, `model/`, `repository/`, `utils/`, `factory/`, `interface/`. Entrypoints: `src/index.ts`, `src/cubicler.ts`.

# Repository Guidelines

## Project Structure & Module Organization

- Source: `src/` — core modules in `core/` (services), `transport/` (`agent/`, `mcp/`), `middleware/`, `model/`, `repository/`, `utils/`, `factory/`, `interface/`. Entrypoints: `src/index.ts`, `src/cubicler.ts`.
- Tests: `tests/` — unit/integration tests mirror `src/` (e.g., `core/`, `repository/`, `utils/`); integration samples in `tests/integration/`.
- Docs: `docs/` — agent, provider, webhook integration guides.
- Build output: `dist/` (via tsup). Config: `agents.json`, `providers.json`, `webhooks.json`, `.env` (see `.env.example`).

## Build, Test, and Development Commands

```bash
npm run dev           # Start in dev (ts-node)
npm run dev:watch     # Dev with watch
npm run build         # Build to dist (tsup)
npm start             # Run built server
npm test              # Vitest (watch); UI via test:ui
npm run test:run      # One-off test run (CI)
npm run lint          # ESLint (TypeScript rules)
npm run lint:fix      # Autofix lint issues
npm run format        # Prettier write
npm run format:check  # Prettier check only
npm run typecheck     # tsc --noEmit
```

Optional Docker: `npm run docker:dev` (compose dev), `npm run docker:prod` (compose up -d).

## Coding Style & Naming Conventions

- TypeScript (Node 18+, ESM). Prettier: 2 spaces, single quotes, `printWidth` 100, trailing commas (es5).
- ESLint + `@typescript-eslint` + Prettier. No unused vars (underscore-ignored), `prefer-const`, `no-var`, sorted imports.
- Names: Files kebab-case (e.g., `provider-rest-service.ts`); functions/vars camelCase; classes/types/interfaces PascalCase.

## Testing Guidelines

- Framework: Vitest (Node env). Place tests under `tests/`, mirroring `src/`; use `*.test.ts` or `*.spec.ts`. Integration tests live in `tests/integration/` and run with the same commands.
- Add tests for every feature/bugfix. Use `tests/mocks/` for fixtures. Ensure `npm run test:run` passes before PRs.

## Commit & Pull Request Guidelines

- Commits: Conventional Commits (e.g., `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`). Keep subjects imperative and concise.
- PRs: include purpose, linked issues, test notes, and config/docs updates; screenshots/logs when behavior changes.
- Preflight: `npm run lint && npm run format:check && npm run typecheck && npm run test:run && npm run build`.

## Security & Configuration Tips

- Never commit secrets. Use `.env`; reference via `${VAR}` or `{{env.VAR}}` in JSON configs.
- Local sanity: `npm run dev`, then check `GET /health` at `http://localhost:1503/health`.
- JWT: see `src/middleware/jwt-middleware.ts` and README’s security section when securing providers/agents.

## Agent-Specific Instructions

- Add a transport/provider: implement interfaces in `src/interface/` (e.g., `agent-transport.ts`, `mcp-transport.ts`), then wire factories in `src/factory/agent-transport-factory.ts` or `src/factory/mcp-transport-factory.ts`.
- Extend services in `src/core/` (dispatch, MCP/provider, webhook) and update models/types in `src/model/`.
- Update example configs in `agents.json`/`providers.json` and add tests under `tests/` mirroring the changed module.

## Engineering Principles

- Architecture: contract-based, SOLID, and dependency inversion; avoid clever hacks.
- Code: TypeScript-first with explicit types; prefer small, pure functions; justify any mutation.
- Errors: never swallow; bubble via typed errors or a Result/Either style.
- Tests: TDD-leaning; propose and add unit tests first (Vitest by default).
- Docs: write short JSDoc for public functions; add an ADR note when changing architecture.
- Security: validate inputs and sanitize external data; never commit secrets.
- Git: make atomic commits with clear messages; show planned diff before writing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cubicler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
