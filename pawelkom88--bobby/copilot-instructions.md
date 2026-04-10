## bobby

> - `app/` holds the Next.js App Router pages, layouts, and route handlers.

# Repository Guidelines

## Project Structure & Module Organization

- `app/` holds the Next.js App Router pages, layouts, and route handlers.
- `components/` contains reusable UI components (PascalCase files like `AchievementBadge.tsx`).
- `context/`, `hooks/`, `lib/`, and `utils/` store shared state, hooks, service clients, and helpers.
- `schemas/` and `types/` define Zod schemas and TypeScript types.
- `messages/` and `i18n/` manage localization content and helpers.
- `__tests__/` contains unit, integration, and e2e suites (see subfolders like `__tests__/e2e`).
- `public/` stores static assets; `scripts/` contains one-off maintenance tools.

## Build, Test, and Development Commands

- `pnpm dev`: start the Next.js dev server on `http://localhost:3000`.
- `pnpm build` / `pnpm start`: build and run the production server.
- `pnpm lint`: run ESLint across the repo.
- `pnpm format`: format with Prettier using repo settings.
- `pnpm test`: run Vitest in CI mode.
- `pnpm exec playwright test`: run Playwright e2e tests (uses `playwright.config.ts`).
- `pnpm emulator`: start Firebase emulators with persisted data; see `emulator:*` scripts for cleanup.

## Coding Style & Naming Conventions

- TypeScript + ESM; keep files in `.ts`/`.tsx`.
- Prettier rules: 2-space indentation, single quotes, semicolons, 80-char lines.
- Components use `PascalCase.tsx`; hooks use `useThing.ts`; helpers use `camelCase`.
- CSS Modules use `*.module.css` alongside components where needed.
- When extracting logic into functions, move them into an existing helper module (or a new helper file) and add unit tests.
- Prefer simple `if` statements with early returns for error handling and message extraction. Avoid nested ternaries, especially patterns like `error instanceof Error ? ... : ...`.

## Testing Guidelines

- Unit/integration tests live in `__tests__/unit` and `__tests__/integration` and use Vitest.
- E2E tests live in `__tests__/e2e` and run with Playwright.
- Name tests `*.test.ts`/`*.test.tsx` and keep fixtures/mocks in `__tests__/fixtures`/`__tests__/mocks`.

## Commit & Pull Request Guidelines

- Follow Conventional Commits as seen in history: `feat:`, `fix:`, `refactor:`, `test:` (optional scope).
- PRs should include a clear description, testing notes (commands + results), and linked issues.
- For UI changes, add screenshots or short clips; for config changes, call out env vars touched.

## Configuration & Security Notes

- Store secrets in `.env.local` and keep them out of commits.
- Firebase configuration lives in `firebase.json`, `firestore.rules`, and `storage.rules`; prefer emulators for local work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pawelkom88)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pawelkom88)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
