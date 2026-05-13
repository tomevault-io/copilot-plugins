## opencode-viewer

> Next.js 15 routes live in `src/app`, and each route keeps its hooks, services, and components in the same folder for small, reviewable slices. The Hono-powered backend and SSE helpers reside in `src/server/service`, while shared Zod schemas stay under `src/lib/conversation-schema` to keep types in sync. UI primitives belong in `src/components/ui`, static assets in `public/`, and the bundled CLI output lands in `dist/`. Co-locate tests beside source files as `*.test.ts(x)` and reuse fixtures from `src/test-setups/`.

# Repository Guidelines

## Project Structure & Module Organization
Next.js 15 routes live in `src/app`, and each route keeps its hooks, services, and components in the same folder for small, reviewable slices. The Hono-powered backend and SSE helpers reside in `src/server/service`, while shared Zod schemas stay under `src/lib/conversation-schema` to keep types in sync. UI primitives belong in `src/components/ui`, static assets in `public/`, and the bundled CLI output lands in `dist/`. Co-locate tests beside source files as `*.test.ts(x)` and reuse fixtures from `src/test-setups/`.

## Build, Test, and Development Commands
Run `pnpm install` (Node 20.12+) before development. Use `pnpm dev` to boot Turbopack with the integrated Hono API at http://localhost:3400. `pnpm build` executes `scripts/build.sh` to assemble the CLI in `dist/`, and `pnpm start` verifies the bundled binary. Quality gates: `pnpm lint` / `pnpm fix` for Biome, `pnpm typecheck` for `tsc --noEmit`, and `pnpm test` or `pnpm test:watch` for Vitest suites.

## Coding Style & Naming Conventions
Biome enforces two-space indentation, double quotes, and sorted imports—run it before committing. Follow `PascalCase.tsx` for components, `camelCase.ts` for utilities, `SCREAMING_SNAKE_CASE` for constants, and suffix atomic UI pieces with `Atom`. Prefer type inference from shared Zod schemas and wire TanStack Query data with Suspense-ready patterns.

## Testing Guidelines
Vitest 3 provides the unit and integration harness configured in `vitest.config.ts`. Mirror production behaviors, assert SSE side effects, and cover failure paths. Name specs `feature.test.ts` or `Component.test.tsx`, keep mocks in `src/test-setups/`, and run `pnpm test` plus `pnpm typecheck` before opening a PR. Capture CLI snapshots whenever observable output changes.

## Commit & Pull Request Guidelines
Use Conventional Commits (e.g., `feat: add session timeline`) and reference issues like `#123` when applicable. PRs should list executed commands, summarize user-facing impact, and attach screenshots or terminal snippets for UI or CLI changes. Document new environment variables or migrations, and request review only after lint, type, and test checks succeed.

## Security & Configuration Tips
Default storage path follows `~/.local/share/opencode/storage/`; adjust via env vars when testing against separate sandboxes. Disable automatic browser launch with `CC_VIEWER_NO_AUTO_OPEN=1` (or `NO_AUTO_OPEN=1`) to keep headless runs quiet. Guard sensitive session data by avoiding commits of files under that storage directory.

---
> Source: [nogataka/opencode-viewer](https://github.com/nogataka/opencode-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
