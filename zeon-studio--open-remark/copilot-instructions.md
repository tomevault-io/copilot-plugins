## open-remark

> This file provides guidance to agentic tools when working with code in this repository.

# CLAUDE.md

This file provides guidance to agentic tools when working with code in this repository.

## Project

OpenRemark — self-hostable comment system for static sites. Next.js 16 (App Router) server + vanilla-TS shadow-DOM embed widget + PostgreSQL via Prisma.

Status: beta. APIs and schema may change without migration paths.

## Commands

Package manager: **pnpm** (v11). Most contributor docs say `yarn` — substitute `pnpm` (or `npm run`).

```bash
pnpm dev              # prisma generate + widget build + next dev (turbopack)
pnpm build            # prisma generate + widget build + next build
pnpm start            # production next start
pnpm typecheck        # tsc --noEmit (run before declaring work done)
pnpm lint             # eslint
pnpm format           # prettier --write "**/*.{ts,tsx}"
pnpm format:check     # prettier --check

# Database (PostgreSQL, Prisma)
pnpm db:migrate       # prisma migrate dev — create + apply
pnpm db:generate      # regenerate Prisma client (output: generated/prisma)
pnpm db:seed          # tsx prisma/seed.ts — demo data
pnpm db:studio        # Prisma Studio GUI
pnpm db:status        # scripts/db-manager.ts status
pnpm db:reset         # DESTRUCTIVE — requires typed confirmation
pnpm db:benchmark:seed
pnpm db:benchmark:run

# Widget bundle
pnpm widget:build     # minified → public/embed.js (+ debug → public/embed.debug.js)
pnpm widget:dev       # non-minified, faster
```

No test runner configured. `pnpm dev` regenerates Prisma client and rebuilds the widget every start — don't skip it.

## Architecture

### Two auth systems, kept separate

| System | Who      | Mechanism                                  | Storage          | Routes                    |
| ------ | -------- | ------------------------------------------ | ---------------- | ------------------------- |
| Admin  | Owners   | Auth.js v5 + Google OAuth → session cookie | Server session   | `/dashboard`, `/api/v1/*` |
| Widget | Visitors | Google `id_token` → signed Widget JWT      | `sessionStorage` | `/api/widget/*`           |

- Admin routes gated by `proxy.ts` (Next middleware — named `proxy.ts`, not `middleware.ts`; matcher: `/dashboard/:path*`). Uses edge-safe `lib/auth.config.ts` (no Prisma adapter).
- Widget uses `Authorization: Bearer <jwt>` — no cookies (cross-origin). CSRF not applicable; CORS allowlist enforced per `Site.allowedOrigins`.
- `WIDGET_JWT_SECRET` is distinct from `AUTH_SECRET` — do not conflate.

### Layered request flow (HARD RULE)

```
app/api/**          route handler  — thin: parse, auth, call service, respond (≤25 lines)
  ↓
lib/services/**     business logic — NO Next.js imports (must be framework-agnostic)
  ↓
lib/db.ts           Prisma singleton — ONLY services may import this
  ↓
PostgreSQL (Prisma adapter-pg)
```

- Route handlers must not import `lib/db.ts` directly. They call services.
- Zod schemas live in `lib/validators/` and are shared by routes + services.
- Response/error helpers in `lib/api/`.

### Directory map (non-obvious only)

- `app/(auth)/` — sign-in route group (no path prefix)
- `app/(dashboard)/` — dashboard layout group; actual pages under `app/dashboard/`
- `app/api/v1/` — admin REST API (session-gated)
- `app/api/widget/` — public widget API (CORS-open, JWT-gated for writes)
- `app/api/auth/[...nextauth]/` — Auth.js handler
- `lib/services/` — `comment-service.ts`, `comment-client.ts`, `moderation-service.ts`, `page-service.ts`, `site-service.ts`, `user-service.ts`
- `lib/auth.ts` vs `lib/auth.config.ts` — full config (with Prisma adapter) vs edge-safe subset for middleware
- `lib/auth-widget.ts` — Widget JWT issue/verify (jose)
- `lib/cors.ts`, `lib/rate-limit.ts` (in-memory LRU, 10 req/min), `lib/sanitize.ts`
- `generated/prisma/` — Prisma client output. Import from `@/generated/prisma/client`. Do not edit.
- `widget/src/` — `index.ts` (entry), `api.ts`, `auth.ts`, `render.ts`, `styles.css`, `types.ts`. Builds via `widget/build.ts` → `public/embed.js`.
- `proxy.ts` (root) — Next middleware. Renamed from `middleware.ts`.
- `prisma.config.ts` — Prisma schema/migrations config.
- `config/config.json`, `.agents/instructions/*.md` — agent-facing rules (see below).

### Widget specifics

- Shadow DOM — CSS isolated. Edit `widget/src/styles.css` for visuals.
- Compile-time injected constants via esbuild `define`: `__APP_URL__`, `__GOOGLE_CLIENT_ID__`, `__STYLES__`. **Do not** read from `process.env` in widget source.
- `widget/build.ts` loads `.env` then `.env.local` before bundling, so widget needs `NEXT_PUBLIC_APP_URL` + `GOOGLE_CLIENT_ID` at build time.
- Auth flow: popup → `accounts.google.com` → redirect to `/api/widget/oauth-callback` → `postMessage({type: "ZEON_GOOGLE_TOKEN", idToken})` → `POST /api/widget/auth` → server verifies via Google tokeninfo → issues 7-day Widget JWT.

### Data model

```
User ──< Site ──< Page ──< Comment ──< Comment (replies, self-ref via @relation("Replies"))
                              │
                              └──< ModerationLog
```

- `Comment.status`: PENDING | APPROVED | SPAM | DELETED (soft delete only — never physically delete domain rows).
- `Site.siteKey` — public embed identifier. `Site.allowedOrigins` is JSON (Postgres native), enforced on widget POST.

## Schema / migration rules (from `.agents/instructions/database-schema-and-migrations.md`)

Follow this file when touching `prisma/schema.prisma`. Highlights:

- Always CUID (`@id @default(cuid())`) for PKs. Never autoincrement int.
- Every domain model: `createdAt` (`@default(now())`) + `updatedAt` (`@updatedAt`).
- `Json` type for JSON columns — not stringified.
- Auth.js models (`User` / `Account` / `Session` / `VerificationToken`) — do not modify structure.
- Soft-delete domain data via status enums; `onDelete: Cascade` only for clearly child rows.
- **Index every FK, every common status+timestamp filter, every WHERE lookup field.** Composite-index column order MUST match query order.
- Workflow: edit schema → `npx prisma validate` → `pnpm db:migrate --name <descriptive>` → review generated SQL → `pnpm db:generate` → verify in app.
- Never edit applied migrations; create a new one.

## Code conventions (from `.agents/instructions/code-guidelines.md`)

- TypeScript strict; no `any`, no unjustified `ts-ignore`.
- Comments: only WHY (hidden constraints / non-obvious invariants), not WHAT.
- Functional style preferred (`map`/`filter`/`reduce` over mutation).
- shadcn/ui for dashboard UI — do not import Radix primitives directly. Tailwind v4 utilities only; no per-component custom CSS.
- Optimistic UI uses `hooks/use-optimistic-state.ts`.
- If logic is needed in 2+ places, extract to `lib/` or `lib/utils.ts` — don't duplicate.

## Environment

Required env (see `.env.example`): `DATABASE_URL`, `AUTH_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `WIDGET_JWT_SECRET`, `NEXT_PUBLIC_APP_URL`.

Google Cloud Console — Authorized redirect URIs must include:

- `<APP_URL>/api/auth/callback/google` (admin Auth.js)
- `<APP_URL>/api/widget/oauth-callback` (widget visitor flow)

---
> Source: [zeon-studio/open-remark](https://github.com/zeon-studio/open-remark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
