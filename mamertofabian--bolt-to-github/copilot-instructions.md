## bolt-to-github

> This is a Chrome Manifest V3 extension built with Svelte, TypeScript, TailwindCSS, Vite, and CRXJS. Source code lives in `src/`: `background/` is the service worker, `content/` integrates with bolt.new, `popup/` contains the extension UI, `pages/` contains standalone extension pages, `services/` handles GitHub/auth/sync logic, and `lib/` contains shared utilities, stores, constants, and components. Tests are colocated in `__tests__/` folders or named `*.test.ts` / `*.spec.ts`. Shared test setup is in `src/test/setup/`; e2e tests are in `e2e/`, assets in `assets/`, and reference docs in `docs/`.

# Repository Guidelines

## Project Structure & Module Organization

This is a Chrome Manifest V3 extension built with Svelte, TypeScript, TailwindCSS, Vite, and CRXJS. Source code lives in `src/`: `background/` is the service worker, `content/` integrates with bolt.new, `popup/` contains the extension UI, `pages/` contains standalone extension pages, `services/` handles GitHub/auth/sync logic, and `lib/` contains shared utilities, stores, constants, and components. Tests are colocated in `__tests__/` folders or named `*.test.ts` / `*.spec.ts`. Shared test setup is in `src/test/setup/`; e2e tests are in `e2e/`, assets in `assets/`, and reference docs in `docs/`.

## Build, Test, and Development Commands

Use `pnpm`; do not switch package managers.

- `pnpm install` installs dependencies from `pnpm-lock.yaml`.
- `pnpm dev` starts the Vite dev server for UI iteration.
- `pnpm watch` builds the extension to `dist/` and watches for changes; load `dist/` in Chrome developer mode.
- `pnpm build` creates a production extension build.
- `pnpm lint` runs ESLint over `src/**/*.{js,ts,svelte}`.
- `pnpm check` runs `svelte-check` with TypeScript validation.
- `pnpm test:ci` runs Vitest once without coverage; `pnpm test` runs Vitest with coverage.
- `pnpm test:e2e` runs Playwright tests.

## Coding Style & Naming Conventions

Use strict TypeScript and ESM imports. Svelte components must use `<script lang="ts">`. Prefer `$lib/*` for imports from `src/lib`. Prettier uses 2 spaces, single quotes, 100-character lines, and ES5 trailing commas. Use TailwindCSS for UI styling and `lucide-svelte` for icons. Avoid `any`; use concrete types or `unknown` with narrowing. Test files should use names such as `FileChanges.component.test.ts`, `ServiceName.test.ts`, or `Feature.logic.test.ts`.

## Testing Guidelines

Vitest uses `jsdom`, globals, and setup from `src/test/setup/vitest-setup.ts`; coverage uses V8 and writes to `coverage/`. Follow `docs/unit-testing-rules.md` for classes, services, and utilities, and `docs/component-testing-rules.md` for Svelte UI tests. Prefer TDD for behavior changes: add a focused failing test, implement the fix, then run the relevant tests plus `pnpm lint` and `pnpm check`.

## Commit & Pull Request Guidelines

History follows Conventional Commits, for example `feat: mint independent Supabase session` and `fix: address PR review issues`. Branch from the active `dev-vX.Y.Z` branch; reserve `main` for releases. PRs should target the active dev branch, describe behavior changes, list validation commands, link issues when applicable, and include screenshots or recordings for visible UI changes.

## Security & Configuration Tips

Never commit GitHub tokens, Supabase secrets, API keys, or generated credential files. Keep Chrome extension permissions minimal and update `manifest.json` deliberately. Make degraded auth, sync, and network states visible in UI or logs.

---
> Source: [mamertofabian/bolt-to-github](https://github.com/mamertofabian/bolt-to-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
