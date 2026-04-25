## nextjs-supabase-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
pnpm dev                    # Start Next.js dev server
pnpm build                  # Production build
pnpm start:all              # Run app + DB studio concurrently

# Database
pnpm db:push                # Push schema to database
pnpm db:migrate <name>      # Generate migration
pnpm db:update              # Apply pending migrations
pnpm db:studio              # Open Drizzle Studio

# Quality
pnpm lint                   # ESLint
pnpm lint:fix               # Auto-fix linting issues
pnpm format                 # Prettier formatting

# Testing
pnpm test:unit              # Vitest unit tests
pnpm test:e2e               # Playwright E2E (all browsers)
pnpm test:e2e:ui            # Playwright with UI
pnpm test:e2e:debug         # Playwright debug mode

# Documentation
pnpm docs:dev               # VitePress dev server (port 4000)
```

## Architecture

### Route Groups
- `app/(public)/` — Public landing page
- `app/(auth)/` — Login, register, forgot/reset password
- `app/(protected)/` — Auth-gated routes (dashboard)
- `app/api/` — API routes

### Server/Client Page Pattern
Pages are split into two files:
- `page.tsx` — Thin server entry point; exports metadata only
- `page.client.tsx` — All interactive logic, forms, and state

### Auth Flow
1. Supabase handles session cookies via `@supabase/ssr`
2. `proxy.ts` (Next.js middleware) refreshes sessions and redirects unauthenticated requests to `/login?redirect=<path>` using `PROTECTED_ROUTE_PATTERNS`
3. API routes use `requireAuth()` from `lib/guards/auth.guard.ts` which returns `{ user, error }`
4. The `useAuth()` hook (`hooks/use-auth.ts`) combines Supabase auth state + user profile from React Query

### Data Layer
- **Drizzle ORM** (`lib/drizzle/db.ts`) — postgres.js driver with connection pooling (`max: 10`, `prepare: false`)
- **Schemas** in `drizzle/schemas/` — all tables extend base columns (`id`, `createdAt`, `updatedAt`, `deletedAt`)
- **Supabase clients** — `lib/supabase/client.ts` (browser singleton), `lib/supabase/server.ts` (async, per-request)

### API Patterns
- Responses use `lib/response.ts` helpers: `{ success: true, data }` / `{ success: false, error }`
- HTTP status codes come from `constants/http-status.constant.ts` (`HttpStatus` object)
- Rate limiting via `lib/ratelimit.ts` (Upstash Redis) — tiers: `api` (20/10s), `auth` (5/60s), `email` (3/60s); gracefully skips when env vars are absent

### React Query Setup
- Query client config: stale time 5 min, GC time 5 min, retry 1, no refetch on window focus
- Query keys via `lib/query/get-query-keys.ts` (`getQueryKey.users.me()`, etc.)
- Query option factories in `queries/` (e.g., `queries/user.query.ts`)

### Component Hierarchy
- `components/ui/` — Shadcn/Radix primitives — **do not modify**
- `components/shared/` — Reusable wrappers around `ui/`
- `components/providers/` — React context providers

## Conventions

### File Naming (kebab-case throughout)
- Components: `component-name.tsx`
- Hooks: `use-hook-name.ts`
- Services: `name.service.ts`
- Query factories: `name.query.ts`
- Constants: `name.constant.ts`
- Zod/Drizzle schemas: `name.schema.ts`

### Constants
- Routes: `constants/routes.constant.ts` — use `AUTH_ROUTES`, `PROTECTED_ROUTES`, `API_ROUTES`
- Sidebar nav: `constants/app-sidebar-items.constant.ts`
- SEO defaults: `constants/seo.constant.ts`

### Error Handling
- Client-side: `useTransition` + toast notifications (sonner)
- API routes: structured JSON with proper HTTP status codes
- Services: try/catch returning `null` on failure

## Environment Variables

```bash
# Required
DATABASE_URL=                          # Supabase PostgreSQL connection
NEXT_PUBLIC_SUPABASE_URL=              # Supabase project URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=  # Public API key

# Optional
NEXT_PUBLIC_SITE_URL=                  # Defaults to http://localhost:3000
RESEND_API_KEY=                        # Email delivery
RESEND_EMAIL_FROM=                     # Sender address
UPSTASH_REDIS_REST_URL=                # Rate limiting
UPSTASH_REDIS_REST_TOKEN=
```

## TypeScript
- Path alias `@/*` maps to the project root
- Strict mode enabled; avoid `any`
- Drizzle inferred types: `SelectProfile`, `InsertProfile` from `types/drizzle.types.ts`
- Environment types declared in `types/globals/env.d.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WannaCry081) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
