## metatell-ai-bot

> - Monorepo managed by `pnpm`. Packages live under `packages/`:

# Repository Guidelines

## Project Structure & Module Organization
- Monorepo managed by `pnpm`. Packages live under `packages/`:
  - `packages/core` (shared services), `packages/sdk` (client SDK), `packages/realtime` (LiveKit/RTC), `packages/cli` (developer CLI).
- Examples in `examples/*` (e.g., `examples/voice-bot`, `examples/basic-bot`).
- Tests colocated with code as `*.spec.ts` under `packages/*/src`. Shared helpers in `test-utils/`.
- Docs in `docs/`. Type configs in `tsconfig*.json`.

## Build, Test, and Development Commands
- Install: `pnpm install` (Node 22 pinned via .node-version).
- Build all: `pnpm build` (recurses). Per package: `pnpm -F @metatell/bot-core build`.
- Typecheck: `pnpm typecheck` or `pnpm -F <pkg> typecheck`.
- Test: `pnpm test` (Vitest). Coverage: `pnpm test:coverage`. UI: `pnpm test:ui`.
- Lint/format: `pnpm check` (Biome). Autofix: `pnpm check:fix`.
- Clean: `pnpm clean`. Docs: `pnpm typedoc`.

## Coding Style & Naming Conventions
- Language: TypeScript with `strict` settings (`tsconfig.base.json`).
- Formatting: Biome (2 spaces, single quotes, trailing commas, no unnecessary semicolons).
- Naming: PascalCase classes, camelCase functions/vars, kebab-case files and package names. Keep cohesive modules under `src/`.

## Testing Guidelines
- Framework: Vitest. Place tests next to code as `*.spec.ts`.
- Coverage via V8; run `pnpm test:coverage` (text/html/json-summary).
- Reuse utilities in `test-utils/` and existing mocks within each package.

## Commit & Pull Request Guidelines
- Always run `pnpm run check` and `pnpm run test` before committing.
- Conventional Commits (`feat:`, `fix:`, `chore:` …). Include motivation and scope.
- Use Changesets for user-facing changes: `pnpm changeset`; version with `pnpm bumpup`; release via `pnpm release`.
- PRs include: clear description, linked issues, tests, and docs updates (package READMEs or `docs/*`). Add screenshots/terminal output for CLI changes.

## Security & Configuration Tips
- Never commit secrets. Use `.env` files in examples (`examples/*/.env.example`).
- Validate external services (e.g., LiveKit, Google APIs) in examples and document required env vars.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/urth-inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
