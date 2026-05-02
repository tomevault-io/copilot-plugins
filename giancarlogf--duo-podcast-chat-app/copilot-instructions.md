## duo-podcast-chat-app

> - Project: Next.js 16 + React 19 + TypeScript app for English practice through stories and phrasal verbs.

# Podcast Chat App - AGENTS Guide

## Purpose
- Project: Next.js 16 + React 19 + TypeScript app for English practice through stories and phrasal verbs.
- Main data/services: MongoDB, Firebase Auth (Admin session cookies), Firestore (`phrasal_verbs`), Gemini.
- Package manager: use `pnpm` only.
- UX copy rule: default to Spanish unless a screen is intentionally English-first.

## Rule Sources (Cursor and Copilot)
- Primary rules source: this `AGENTS.md` file.
- Cursor rules audit:
  - `.cursorrules`: not found.
  - `.cursor/rules/`: not found.
- Copilot rules audit:
  - `.github/copilot-instructions.md`: not found.
- If any of those files are added later, merge their rules here and prioritize the most specific scope.

## Build, Lint, Typecheck, Test Commands
- Install deps: `pnpm install`
- Dev server: `pnpm dev`
- Production build: `pnpm build`
- Production start: `pnpm start`
- Lint full repo: `pnpm lint`
- Lint + auto-fix: `pnpm lint --fix`
- Lint one file/folder: `pnpm lint -- src/features/stories/presentation/actions.ts`
- Type-check (required): `pnpm exec tsc --noEmit`

### Test Commands (Current Repo State)
- Current state:
  - No `test` script in `package.json`.
  - No Jest/Vitest/Playwright config found.
  - No `*.test.*` or `*.spec.*` files found.
- If Vitest gets added:
  - Run all: `pnpm vitest run`
  - Watch mode: `pnpm vitest`
  - Single file: `pnpm vitest run path/to/file.test.ts`
  - Single test name: `pnpm vitest run path/to/file.test.ts -t "test name"`
- If Playwright gets added:
  - All e2e: `pnpm playwright test`
  - Single spec: `pnpm playwright test tests/e2e/example.spec.ts`
  - Single test name: `pnpm playwright test tests/e2e/example.spec.ts -g "name"`

## Architecture Map
- App Router root: `src/app`
- Feature route groups:
  - `src/app/(features)/stories/**`
  - `src/app/(features)/phrasal-verbs/**`
- API routes: `src/app/api/**/route.ts`
- Feature layers: `domain`, `application`, `infrastructure`, `presentation`
- Shared cross-feature modules: `src/shared/**`
- Scripts: `scripts/*.mjs`

## Imports and Boundaries
- Prefer alias imports over deep relative paths.
- Active aliases from `tsconfig.json`:
  - `@/*`
  - `@/features/*`
  - `@/shared/*`
- Import ordering:
  1) framework/third-party
  2) alias imports
  3) same-folder relative imports
- Use `import type` for type-only imports.
- Keep layer boundaries clean; avoid presentation leakage into infrastructure.
- Avoid circular dependencies between features/layers.

## Formatting and Styling
- Preserve file-local style and avoid unrelated reformatting.
- Semicolon style is mixed across the repo; match the file you edit.
- Quote style is mixed; match the file you edit.
- Keep Tailwind class strings stable; avoid class churn without behavior/UI change.
- Use `cn` from `@/shared/presentation/utils` for conditional class composition.
- Add comments only for non-obvious logic.

## TypeScript Rules
- `strict: true` is enabled and must be respected.
- `allowJs: true` exists, but new code should be TypeScript-first.
- Prefer explicit return types on exported functions.
- Avoid `any`; use domain entities, interfaces, DTOs, and narrow unions.
- Convert Mongo `_id` to strings before sending data to client boundaries.
- Use schema validation (for example `zod`) where validation patterns already exist.
- Important: `next.config.mjs` sets `ignoreBuildErrors: true`; `pnpm build` is not a type gate.

## Naming Conventions
- Components: PascalCase (`EpisodeCard`).
- Hooks: `useX` (`useUserSession`).
- Use cases: verb-oriented PascalCase filenames (`GetStories.usecase.ts`).
- Server actions: descriptive camelCase (`startChatByEpisode`).
- Types/interfaces/entities: PascalCase with domain meaning.
- Constants: `UPPER_SNAKE_CASE` only for real constants.
- Keep filenames aligned with existing feature patterns.

## Next.js / React Conventions
- Add `'use client'` only when browser APIs/hooks are required.
- Keep server components hook-free.
- Server actions live in feature `presentation/actions.ts` with `'use server'`.
- Use `revalidatePath` when mutations affect cached server-rendered views.
- For query-driven screens, parse in server pages and pass typed props to children.
- On optimistic UI, include explicit rollback/error handling.

## Authentication (Firebase Session Cookie Model)
- Session cookie name: `__session`.
- Login flow: Firebase client auth -> ID token -> `POST /api/auth/session-login` -> Admin session cookie.
- Session verification for server rendering/actions occurs via Admin SDK (`verifySessionCookie`).
- Logout flow: `POST /api/auth/session-logout` clears cookie and attempts refresh-token revocation.
- Session TTL configured in `src/shared/infrastructure/auth/session.ts` (`SESSION_MAX_AGE_SECONDS`, currently 7 days).

## Data Access Rules
- MongoDB:
  - Always call `await dbConnect()` before model operations.
  - Use model getter helpers in `src/shared/infrastructure/database/mongo/models/*`.
  - Avoid ad-hoc `mongoose.model(...)` declarations.
  - For `_id` queries, convert strings using `new mongoose.Types.ObjectId(...)`.
- Firestore:
  - Collection for verbs is `phrasal_verbs`.
  - Normalize filter/taxonomy strings (trim/case normalization) before matching.

## Error Handling and Logging
- Validate required inputs early and return clear client-facing errors.
- API status conventions:
  - `400` for invalid client input
  - `500` for unexpected infrastructure/runtime failures
- Prefer consistent JSON payloads: `success`, `error`, optional `details`.
- Include operation context in logs (for example `episodeId`, `userProgressId`).
- Use `console.warn` for recoverable issues and `console.error` for failures.
- Never log secrets, auth tokens, or raw third-party payloads.

## Environment and Secrets
- Expected env vars:
  - `MONGODB_URI`
  - `GEMINI_API_KEY`
  - `NEXT_PUBLIC_FIREBASE_*`
  - `FIREBASE_PROJECT_ID`
  - `FIREBASE_CLIENT_EMAIL`
  - `FIREBASE_PRIVATE_KEY` (escaped `\n`, normalized in code)
- Script usage pattern requiring env file:
  - `node --env-file=.env.local scripts/seed-episodes.mjs`
  - `node --env-file=.env.local scripts/cleanup-episodes.mjs`
  - `node --env-file=.env.local scripts/update-episode-titles.mjs`
- Never commit `.env*` files, tokens, or service-account secrets.

## Quality Gates Before Merge
- Required: `pnpm lint`
- Required: `pnpm exec tsc --noEmit`
- Optional smoke check: `pnpm build`
- Note: build may pass while type-check fails.

## Manual QA Smoke Checklist
- `/stories`: cards and section grouping render correctly.
- Story chat: start episode, submit translation, feedback appears.
- Progress persistence: reload chat URL and verify continuity.
- `/phrasal-verbs`: search/filter/pagination/modal behavior works.
- `/phrasal-verbs/practice`: flow supergroup -> group -> category works.
- `/phrasal-verbs/practice/session`: selected category is consumed and matching verbs appear.

## Quick Checklist for Coding Agents
- Use `pnpm` only.
- Respect existing file-local formatting.
- Prefer alias imports and `import type`.
- Keep strict typing and explicit exported return types.
- Validate inputs and keep API error responses consistent.
- Keep Spanish-first UX copy unless page intent is English-first.

---
> Source: [GiancarloGF/duo-podcast-chat-app](https://github.com/GiancarloGF/duo-podcast-chat-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
