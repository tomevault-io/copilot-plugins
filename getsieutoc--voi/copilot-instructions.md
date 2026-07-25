## voi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`voi` is a feature-voting / public-roadmap app (Canny-style). Users submit
posts, vote on them, and admins move posts through statuses
(`REVIEW → PLANNED → IN_PROGRESS → RELEASED / DISCARDED`).

## Stack

- **Next.js 16** (App Router, Turbopack), **React 18.3**, TypeScript strict, Node `24.x`
- **pnpm 10.20** — required, not interchangeable with npm/yarn
- **Better Auth** with the **Email OTP** plugin (6-digit code, 5-min expiry) + `admin` plugin
- **Prisma 6** with `@prisma/adapter-pg` (driver adapter) on PostgreSQL 16
- shadcn/ui on Radix, Tailwind 3.4, **SWR** (client fetching), **Jotai** (client state),
  react-hook-form + zod (forms)

## Commands

```bash
pnpm dev               # full dev: docker compose up (postgres+mailpit) + next dev --turbo
pnpm dev:next          # next dev --turbo only (use when DB is already up)
pnpm compose up -d     # bring up postgres + mailpit only
pnpm compose stop      # stop dev containers
pnpm typecheck         # tsc --noEmit — THE quality gate (lint is effectively disabled)
pnpm build             # next build
pnpm prisma migrate dev    # apply migrations (dev server must be running)
pnpm prisma generate       # regenerate ./prisma/client (also runs on postinstall)
pnpm ui add <component>    # add a shadcn primitive
```

There is **no test runner configured** — no `test` script, no Jest/Vitest/Playwright
installed. If asked to "run the tests" or to follow strict TDD, flag this and ask
whether to add the toolchain rather than pretending tests exist.

## Architecture

- **`app/`** — App Router. `app/(main)/` is the auth-gated route group (dashboard,
  profile). `app/api/` holds the Better Auth catch-all (`auth/[...all]/route.ts`),
  plus `search/` and `votes/count/`.
- **`components/ui/`** — shadcn primitives. Don't hand-edit; regenerate via
  `pnpm ui add <name>`. **`components/client/`** — feature-level client islands,
  every file is `'use client'`. **`components/icons/`** — icon barrel.
- **`services/`** — Server Actions (`'use server'`): `posts.ts`, `users.ts`,
  `votes.ts`. Each action calls `getSession()` from `@/lib/auth` for authz and
  `revalidatePath()` after mutations. **Mutations live here, not in API routes.**
- **`lib/auth.ts`** — Better Auth server config (OTP + admin plugin) + `getSession()`.
  **`lib/auth-client.ts`** — `authClient` for React (`useSession`, `signIn`,
  `signOut`, `emailOtp`).
- **`lib/prisma.ts`** — Prisma singleton with `globalThis` dev-mode caching.
- **`hooks/`** — barreled custom hooks (`use-auth`, `use-posts`, `use-disclosure`,
  `use-loading`).
- **`prisma/schema.prisma`** — models: `User`, `Account`, `Session`, `Verification`
  (Better Auth tables), `Post`, `Vote`, `Tag`. The `Status` enum drives the roadmap.
- **`emails/`** — React Email templates. Note: the OTP email is currently inline
  HTML in `lib/auth.ts`, so templates here may be unused.

## Quirks that bite

- **Prisma client is generated to `./prisma/client/`**, not `node_modules`.
  Import from `@/prisma/client/client` (types + `PrismaClient`) and
  `@/prisma/client/enums` (`Role`, `Status`). `postinstall` runs `prisma generate`.
- **`pnpm dev` is not just `next dev`** — it runs
  `pnpm compose up -d && pnpm watch && pnpm compose stop`. So `Ctrl+C` tears down
  Postgres + Mailpit. Use `pnpm dev:next` if you want the Next process alone.
- **Mailpit catches all dev emails** at <http://localhost:8025> (SMTP on
  `127.0.0.1:1025`). OTP login codes only appear there in dev — there's no
  console fallback.
- **First user auto-promoted to ADMIN** via `databaseHooks.user.create.before`
  in `lib/auth.ts`. After a DB reset, whoever signs in first is admin.
- **Auth cookies are prefixed `voi`** (Better Auth `advanced.cookiePrefix`),
  e.g. `voi.session_token`.
- **Path alias** `@/*` → repo root (`tsconfig.json`). `@/lib/...`, `@/components/...`,
  `@/prisma/client/...`.
- **Linting is effectively disabled.** `pnpm lint` exists but two eslint configs
  coexist (`eslint.config.js` + `.mjs`) and the project rule says quality gating
  runs `pnpm typecheck` only.
- **Avoid `useEffect`** (`.cursor/rules/avoid-use-effect.mdc`). Reach for Server
  Actions, SWR, event handlers, refs, or `useSyncExternalStore` first.
- **Server Components by default.** `page.tsx` files stay server components;
  client islands go in `components/client/` with `'use client'`.

## Conventions enforced by tooling

- **Conventional Commits required.** `commitlint` + `husky` reject malformed
  commit messages. Types: `feat`, `fix`, `perf`, `refactor`, `docs`, `style`,
  `test`, `chore`, `ci`, `build`, `revert`.
- **Master pushes auto-release.** `semantic-release` bumps the version, writes
  `CHANGELOG.md`, creates a GitHub release, and publishes a Docker image to
  `ghcr.io/getsieutoc/voi`. Don't hand-edit `CHANGELOG.md` or `package.json`
  version.

## Environment

Required env vars (see `.env.example`):
`BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`, `DATABASE_URL`,
`SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`.
Optional: `EMAIL_FROM`, `NEXT_PUBLIC_PROJECT_NAME`, `PROJECT_NAME`.

---
> Source: [getsieutoc/voi](https://github.com/getsieutoc/voi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
