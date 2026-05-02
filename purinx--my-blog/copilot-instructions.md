## my-blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # local dev server (Hono + Vite)
pnpm build        # production build to ./dist
pnpm preview      # serve dist with wrangler pages dev
pnpm deploy       # deploy via wrangler
pnpm lint         # lint (oxlint)
pnpm lint:fix     # lint with auto-fix
pnpm format       # format (prettier --write)
pnpm format:check # prettier --check
pnpm check:type   # type-check
pnpm check        # type + lint + format-check
pnpm check:fix    # type + lint-fix + format
pnpm sqlite       # open local D1 sqlite DB (wrangler local state)
```

No test suite exists in this project.

## Architecture

This is a **Hono SSR blog** running on **Cloudflare Pages**, with:

- **`src/index.tsx`** — single entry point; defines all routes using Hono. Routes render JSX directly (no client-side React). API routes under `/api/*` are protected by Bearer/x-api-key auth.
- **`src/db/index.ts`** — defines `Bindings` (D1 + API_KEY) and helper accessor (`getDb`).
- **`src/backend/repositories/postRepository.ts`** — all D1 data access. Post metadata and post content (Markdown) are both stored in D1 (`posts` table).
- **`src/utils/markdown.ts`** — renders Markdown to HTML via unified/remark/rehype pipeline; also extracts a TOC.
- **`src/components/`** — Layout, PostCard, PostDetail (JSX, server-rendered only).

### Storage

- **D1** stores post metadata and raw Markdown content (`content` column). `content_hash` is used as HTTP ETag.

### Deployment

- `wrangler.toml` configures Cloudflare Pages with D1 binding `DB`.
- DB migrations are in `migrations/` and applied via wrangler (`database_id` in wrangler.toml is a placeholder for local dev).
- Vite + `@hono/vite-cloudflare-pages` + `@hono/vite-dev-server` handle local dev and bundling.

## Coding Guideline

### General

- File names must use kebab-case
- Each module must have accompanying unit tests
- Declare functions using the `function` syntax; do not use arrow functions
  - Do not use `this`
- Use `type` declarations for type definitions
  - Keep types DRY by using `Omit` and `Pick` where applicable
- Prefer `named export`s
  - Avoid `default export` unless required by the framework

### Frontend

- Keep component responsibilities small and focused:
  - `src/ui` — UI primitive components
  - `src/features/` — Feature-scoped packages; subdirectories allowed
    - e.g. `components/`, `hooks/`
  - `src/components/` — Container components referenced directly from `src/index.tsx`
- Prefer extracting reusable UI parts into `src/ui` before adding page-specific UI in `src/components/`

### Backend

```
backend/
  controllers/
  repositories/
  domain/
    [model]/
      index.ts   # model class declaration
      ...        # value objects
  utils/
```

- **Controller** — Extract one controller per REST resource; validate requests with Valibot ([docs](https://valibot.dev/api/))
- **Repository** — Encapsulate all DB access at the transaction level in repositories
- **Utils** — Extract shared utilities into `utils/`
- **Async / error handling** — Use Effect for all async operations and error handling in the backend ([docs](https://effect.website/docs))
- Prefer Hono's built-in SSR (pass props directly) over REST API calls where possible

### Domain

- Define data structures that reflect the domain model
- May be used from either frontend or backend
- Define using Effect's `Data.Class`
- Value Objects should also be defined as `Data.Class`

---
> Source: [purinx/my-blog](https://github.com/purinx/my-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
