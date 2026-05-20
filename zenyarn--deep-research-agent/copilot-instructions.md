## deep-research-agent

> This repository is a Next.js App Router application. Route files live in `app/`, including UI pages such as `app/page.tsx` and server endpoints under `app/api/**`. Shared React components are split between `components/core` for product features and `components/ui` for shadcn/ui primitives. Client state, shared types, and utilities live in `lib/` (`lib/store`, `lib/types`, `lib/utils`). Static assets belong in `public/`, and the step-by-step implementation notes in `docs/` describe the intended architecture and feature flow.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Next.js App Router application. Route files live in `app/`, including UI pages such as `app/page.tsx` and server endpoints under `app/api/**`. Shared React components are split between `components/core` for product features and `components/ui` for shadcn/ui primitives. Client state, shared types, and utilities live in `lib/` (`lib/store`, `lib/types`, `lib/utils`). Static assets belong in `public/`, and the step-by-step implementation notes in `docs/` describe the intended architecture and feature flow.

## Build, Test, and Development Commands
- `npm install`: install dependencies.
- `npm run dev`: start the local app with Turbopack.
- `npm run build`: create a production build and catch type or route issues.
- `npm run start`: serve the production build locally.
- `npm run lint`: run Next.js ESLint rules.

Use `npm run lint && npm run build` before opening a PR.

## Coding Style & Naming Conventions
Use TypeScript with strict typing and the `@/*` import alias from `tsconfig.json`. Match the existing code style in each file, which currently uses double quotes, semicolons, and 2-space indentation in most source files.

Use PascalCase for React components (`ResearchReport.tsx`), camelCase for functions and utilities, and descriptive route names under `app/api`. Keep UI logic in components, shared business logic in `app/api/lib` or `lib/`, and avoid duplicating constants across layers. ESLint is configured through `eslint.config.mjs` with `next/core-web-vitals` and `next/typescript`.

## Testing Guidelines
There is no dedicated automated test suite yet. For now, treat `npm run lint` and `npm run build` as required checks, then manually verify key flows in the browser, especially the research stream, API responses, and preview pages like `app/api-test/page.tsx` and `app/components-preview/page.tsx`. When adding tests, colocate them near the feature or add a `tests/` directory and use `*.test.ts` or `*.test.tsx`.

## Commit & Pull Request Guidelines
Recent history uses concise Conventional Commit-style prefixes such as `feat:`, `feat(streaming):`, and `docs:`. Keep commits focused and imperative, for example `feat(api): improve report streaming resilience`. PRs should include a short summary, affected routes/components, verification steps, linked issues, and screenshots or screen recordings for UI changes.

## Security & Configuration Tips
Store secrets in `.env.local` only. The app currently validates `OPENROUTER_API_KEY` and `EXA_SEARCH_API_KEY`; never hardcode them or commit env files. If you change API behavior, update the matching notes in `docs/` so implementation guidance stays in sync.

---
> Source: [zenyarn/deep-research-agent](https://github.com/zenyarn/deep-research-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
