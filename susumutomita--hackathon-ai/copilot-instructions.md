## hackathon-ai

> Source code lives under `src/`, with Next.js routes split between `src/app` (App Router) and `src/pages/api` for server handlers. Shared UI is under `src/components`. Hooks and utilities live in `src/hooks` and `src/lib`. Domain adapters reside in `src/adapters`. Factories and interfaces are in `src/factories` and `src/interfaces`. Tests sit in `**/__tests__` alongside related code, using mocks from `src/__mocks__`. Place static assets in `public/` and keep configuration files (e.g., `tsconfig.json`, `vitest.config.ts`) at the repo root.

# Repository Guidelines

## Project Structure & Module Organization
Source code lives under `src/`, with Next.js routes split between `src/app` (App Router) and `src/pages/api` for server handlers. Shared UI is under `src/components`. Hooks and utilities live in `src/hooks` and `src/lib`. Domain adapters reside in `src/adapters`. Factories and interfaces are in `src/factories` and `src/interfaces`. Tests sit in `**/__tests__` alongside related code, using mocks from `src/__mocks__`. Place static assets in `public/` and keep configuration files (e.g., `tsconfig.json`, `vitest.config.ts`) at the repo root.

## Build, Test, and Development Commands
Use `pnpm install` or `make install` once to set up dependencies. Run `pnpm dev` or `make dev` to start the app at http://localhost:3000. Ship builds with `pnpm build` (or `make build`) and serve production output via `pnpm start`. Execute `pnpm lint`, `pnpm run typecheck`, and `pnpm run format_check` before merging. Run tests with `pnpm test`; add coverage via `pnpm run test:coverage`.

## Coding Style & Naming Conventions
Write all code in TypeScript and let Prettier handle formatting; do not manually adjust spacing or quotes. React components and types use PascalCase, files and directories use kebab-case, and variables/functions use camelCase. Keep modules focused and colocate related tests. Add brief comments only when intent is non-obvious.

## Testing Guidelines
Vitest with the V8 provider is the standard harness. Target ≥90％ coverage and prioritize meaningful assertions over snapshot churn. Name specs `*.test.ts` or `*.test.tsx` and keep them near the implementation (e.g., `src/lib/__tests__/formatter.test.ts`). Use `pnpm test:watch` during development and run `pnpm test:coverage` in CI or before PR submission.

## Commit & Pull Request Guidelines
Follow Conventional Commits, optionally with emoji scopes (e.g., `feat(ui): add idea search (#123)` or `🐛 fix: improve error logging`). Craft PR descriptions that summarize intent, link relevant issues, and include before/after screenshots for UI changes. Ensure the pre-commit sequence passes (`make before_commit`) and note any follow-up work or known limitations in the PR body.

## Security & Configuration Tips
Load secrets through `.env` or `.env.test`, never committing keys such as `QD_API_KEY` or `NOMIC_API_KEY`. Run on Node 18+ with pnpm 8+, and prefer `pnpm run mcp:install` followed by `pnpm run mcp:build` when working with the MCP server. Add new environment variables to `.env.example` and document required setup steps in the PR.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/susumutomita)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/susumutomita)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
