## lore

> Lore is a monorepo centered on `web/`, a Next.js UI/API app. Use `web/app/` for routes, `web/components/` for React components, `web/components/ui/` for shared UI primitives, `web/lib/` for frontend utilities, and `web/server/` for backend code. Tests live in nearby `__tests__/` directories. Integrations live in plugin directories and `pi-extension/`. Assets live in `docker-compose*.yml`, `postgres/`, `scripts/`, and `docs/`.

# Repository Guidelines

## Project Structure & Module Organization

Lore is a monorepo centered on `web/`, a Next.js UI/API app. Use `web/app/` for routes, `web/components/` for React components, `web/components/ui/` for shared UI primitives, `web/lib/` for frontend utilities, and `web/server/` for backend code. Tests live in nearby `__tests__/` directories. Integrations live in plugin directories and `pi-extension/`. Assets live in `docker-compose*.yml`, `postgres/`, `scripts/`, and `docs/`.

## Build, Test, and Development Commands

- `cd web && npm run dev`: start local development.
- `cd web && npm run build`: build production output.
- `cd web && npm run typecheck`: run TypeScript checks.
- `cd web && npm test`: run typecheck plus Vitest.
- `cd pi-extension && npm test`: run Pi extension tests.
- `cd openclaw-plugin && npm run build && npm test`: build and test OpenClaw.
- `docker compose up -d --build`: build and run the full local stack.

## Coding Style & Component Conventions

Write strict TypeScript and prefer named exports. Use two-space indentation, camelCase for variables and functions, PascalCase for React components and types, and snake_case for database columns. Group imports as Node, external, project, then relative.

Business pages import UI from `@/components/ui`: `PageCanvas`, `PageTitle`, `Section`, `Card`, `Button`, form controls, and table helpers. Direct Lobe UI imports stay inside `web/components/ui/` wrappers for repeated patterns, providers, surfaces, and controls. Keep pages focused on data flow and composition.

## Testing Guidelines

Vitest is the test runner. Add tests next to the covered module in `__tests__/` with names like `layout.test.tsx` or `ConfirmDialog.test.tsx`. Bug fixes should include a regression test. Run `cd web && npm test` for `web/` changes; run plugin package tests for integrations.

## Commit & Pull Request Guidelines

Branch from `main` and keep each branch scoped to one change. Use `feat/`, `fix/`, `chore/`, `refactor/`, or `docs/`; separate mechanical refactors from behavior changes. Use Conventional Commits: `feat: add setup step`, `fix: handle empty state`, `chore: update release assets`, `docs: clarify setup`. Pull requests need rationale and test evidence. Include screenshots for visible UI changes.

## Documentation, Release & Configuration

Keep README content focused on installation and configuration. Use GitHub Release notes for changelogs with Highlights, Changes, Upgrade Notes, and Verification. Use prerelease tags like `vX.Y.Z-pre.N` and stable tags like `vX.Y.Z`; release commits should be `release: vX.Y.Z` or `release: vX.Y.Z-pre.N`. Sync versioned package and plugin manifests before tagging. Use `.env.example` as the committed template; keep credentials and local overrides in untracked env files.

---
> Source: [FFatTiger/lore](https://github.com/FFatTiger/lore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
