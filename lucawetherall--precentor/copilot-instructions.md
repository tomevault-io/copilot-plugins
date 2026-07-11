## precentor

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Precentor

Precentor is a planning tool for church musicians: it manages services, the lectionary, music and hymn selection, rotas, and generates printable service sheets. Stack: **Next.js 16 (App Router) · React 19 · Drizzle ORM (Postgres via Supabase) · Tailwind 4 · Zod · Vitest + Playwright**.

The domain vocabulary (rota, collect, lectionary, precentor, canticle…) is **not** general knowledge — read [`docs/glossary.md`](docs/glossary.md) before modelling any domain logic so you don't mis-name or mis-structure things.

## Commands

| Task | Command |
|------|---------|
| Install | `npm ci` |
| Dev server | `npm run dev` |
| **Verify before done** | **`npm run check`** — lint + typecheck + unit tests |
| Unit tests (watch) | `npm run test:watch` |
| E2E tests | `npm run test:e2e` |
| Production build | `npm run build` |
| Apply schema to the DB | `npx drizzle-kit push` — **read [`src/lib/db/AGENTS.md`](src/lib/db/AGENTS.md) first** |

Node 24, **npm only** — there is a `package-lock.json`; do not use yarn/pnpm/bun. Run `npm run check` and make it pass before you claim a task is complete. CI runs the same lint + typecheck + tests (plus build and E2E) and will block the PR otherwise.

## Where things live

| Path | What |
|------|------|
| `src/app/(app)/` | Authenticated app pages — React Server Components by default |
| `src/app/(auth)/` | Login / signup / invite / password pages |
| `src/app/api/` | Route handlers (`route.ts`) |
| `src/components/ui/` | Reusable primitives — `Dialog`, `useToast`, `useConfirm`… |
| `src/lib/auth/` | `requirePageAuth` (pages) · `requireChurchRole` / `requireAuth` (API) |
| `src/lib/api/parse-body.ts` | `parseJsonBody(req, zodSchema)` for request bodies |
| `src/lib/db/` | Drizzle schema, relations, queries — has its own `AGENTS.md` |
| `src/lib/env.ts` | The `env` proxy — import this, **never** read `process.env` directly |
| `src/data/liturgy/` | Hard-coded liturgical content (orders of service) |
| `scripts/` | One-off `tsx` seed / scrape / migrate scripts (`npm run db:*`) |
| `e2e/` | Playwright specs |
| `docs/superpowers/` | Past specs & plans — skim the relevant one before re-designing a feature |

Tests are colocated in `__tests__/` folders next to the code they cover. The import alias `@/*` maps to `src/*`.

## Conventions

Before adding a new page, API route, dialog, **database change**, or anything user-facing, check the relevant checklist in [`docs/conventions.md`](docs/conventions.md). The conventions exist to prevent regressions found in the 2026-05 audit. They are enforced by ESLint where possible (`eslint.config.mjs`, `tools/eslint-rules/`); the doc marks which items are machine-checked vs. left to you.

---
> Source: [lucawetherall/precentor](https://github.com/lucawetherall/precentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
