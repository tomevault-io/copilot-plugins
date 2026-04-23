## agero

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Commands

```bash
npm run dev        # start dev server (Turbopack)
npm run build      # production build
npm run lint       # ESLint (note: build no longer runs lint automatically in Next.js 16)

npx prisma generate          # regenerate Prisma client after schema changes
npx prisma migrate dev       # apply schema changes to the database
npx prisma migrate deploy    # apply migrations in production
npx prisma studio            # open Prisma Studio GUI
```

## Next.js 16 breaking changes

**`middleware.ts` is deprecated.** The request interception file is now `proxy.ts`, and the exported function must be named `proxy` (or a default export). The type is `NextProxy` (which is an alias for the deprecated `NextMiddleware`). The `runtime` config option is not available in proxy files.

Before writing any Next.js code, read the relevant guide in `node_modules/next/dist/docs/`.

## Architecture

### Auth (Clerk)
- `src/proxy.ts` — intercepts all requests; wraps `clerkMiddleware()` from `@clerk/nextjs/server` and exports it as `proxy`
- `src/app/layout.tsx` — root layout wraps the app in `<ClerkProvider>`
- Auth state is read server-side via `@clerk/nextjs/server` helpers (`auth()`, `currentUser()`)

### Database (Prisma + Supabase)
- Prisma schema: `prisma/schema.prisma`; connection URLs live in `prisma.config.ts` (not in the schema file in Prisma 7)
- Prisma v7 uses the `prisma-client` generator; output is `src/generated/prisma` — import types/client from `src/generated/prisma/client` only inside `src/lib/prisma.ts`
- App code should import `prisma` from `src/lib/prisma.ts` (singleton with `@prisma/adapter-pg` + pooled `DATABASE_URL`)
- Supabase: pooled `DATABASE_URL` (transaction pooler, port 6543) at runtime; `DIRECT_URL` (`db.*.supabase.co:5432`) for `prisma migrate` via `prisma.config.ts`
- `src/lib/supabase/server.ts` — service-role client for Server Components, Route Handlers, Server Actions
- `src/lib/supabase/client.ts` — anon-key client for Client Components (real-time, client-side reads only)

### Environment variables
Two env files are used:
- `.env.local` — loaded by Next.js at runtime (Clerk keys, Supabase URL/keys, `DATABASE_URL`)
- `.env` — loaded by `prisma.config.ts` via `dotenv/config` for Prisma CLI commands; must also contain `DATABASE_URL`

### Domain context
- ISO 45001 construction safety platform for Victoria, Australia
- Supabase project: `hgibuaddqimhucbekiuu` — region `ap-southeast-2` (Sydney)
- Roles: `admin` (Director), `safety_manager`, `project_manager`, `site_manager`, `subcontractor_admin`
- RBAC utility: `src/lib/auth.ts` — `requireRole(roles[])`, `getAppUser()`, `AGERO_ROLES`, `ADMIN_MANAGER_ROLES`, `ROLE_LABELS`
- Unauthorized page: `/unauthorized`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bernie-Fernandez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
