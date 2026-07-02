## llm-rules

> - **Monorepo**: pnpm workspaces + Turborepo

# Claude Code Instructions

## Stack

- **Monorepo**: pnpm workspaces + Turborepo
- **Web app**: Next.js 15 (App Router), TypeScript strict, Tailwind CSS v4
- **Auth**: Better Auth (email/password, session cookies)
- **API**: tRPC v11 with `publicProcedure` / `protectedProcedure`
- **DB**: PostgreSQL + Drizzle ORM (`packages/db`)
- **Testing**: Vitest + Testing Library + Playwright (e2e)
- **Logging**: `packages/logger` (structured logging — prefer over `console.log`)
- **i18n**: `packages/i18n` (translations via `next-intl`)

## Architecture Principles

### Server-first rendering

- Pages are Server Components by default. Only add `"use client"` to leaf components that need interactivity — browser APIs, event handlers, or hooks.
- Do server-side data fetching in RSC via `getSession()`, tRPC server caller, or server actions. Pass results down as props — do not re-fetch on the client what was already fetched on the server.
- Protect pages at the RSC level (`getSession()` → redirect) _and_ in middleware. Both layers.

### Small, composable components

- One component = one responsibility. If a component has more than ~150 lines, it's doing too much.
- Prefer pure presentational components that receive props over components that fetch their own data.
- Compose complex UIs from small primitives (`Button`, `Input`, `Card`) rather than building monolithic page components.
- Co-locate page-specific components in `_components/` next to their route. Shared components go in `src/components/`.
- Follow DRY: extract reusable logic, components, and utilities. On the backend, share via tRPC procedures or shared libraries. On the frontend, share via hooks, components, and utility functions.

### Separation of concerns

```
src/
  app/              ← routing, layouts, pages (thin — orchestrate, don't compute)
    _components/    ← page-scoped components (not shared)
  components/ui/    ← design system primitives (stateless, no data fetching)
  server/api/       ← tRPC routers (application logic, auth, validation — call services)
  server/services/  ← service layer (pure business logic, DB access)
  lib/              ← shared utilities, auth config, constants
  db/               ← DB client + re-exports from packages/db
```

- **Pages** call data functions and pass results to components — no raw DB calls in JSX.
- **tRPC routers** handle auth, input validation, and orchestration. They delegate business logic to service functions.
- **Services** contain pure business logic and DB access. They return result types — never throw for expected failure cases.
- **UI components** are purely presentational. They receive data via props.

### Testable code

- Write functions that are pure and side-effect-free wherever possible.
- Inject dependencies (DB, auth, external clients) via parameters — never import them directly inside business logic functions.
- Unit test pure functions and tRPC procedures. Integration test DB interactions with a real connection (use stubs/fakes for external services, never mocks for database queries).
- E2E test all user-facing features with Playwright (`pnpm test:e2e`).
- Place tests alongside source: `foo.test.ts` next to `foo.ts`.

## Conventions

### TypeScript

- Strict mode always. No `any` unless wrapping an untyped third-party boundary.
- Prefer `unknown` + type guards over `any`.
- Use Zod for all external input validation (tRPC inputs, form data, env vars).
- Define types close to where they're used. Only promote to shared if used in 3+ places.
- Use shared constants and enums from `@/lib/constants` for values used across the app (status codes, role names, etc.).

### Components

- Props interfaces above the component, not inlined.
- Use `React.ComponentProps<"div">` spreads for wrapper components.
- No default exports for components in `components/ui/` — named exports only.
- `"use client"` at the top of the file, not buried inside.

### tRPC

- `protectedProcedure` by default — only use `publicProcedure` when there is an explicit need (e.g. public page with public data, login/signup endpoints).
- Input validation with Zod on every mutation.
- Keep routers single-domain (e.g. `usersRouter`, `postsRouter`).
- Routers handle auth, validation, and request orchestration. Delegate business logic to service functions in `server/services/`.

### Styling

- Tailwind utility classes. No inline `style={{}}` unless setting a CSS variable or dynamic value Tailwind can't express.
- Design system: clean white surfaces (`bg-white`), light borders (`border-[#ebebeb]`), subtle shadows, and `#5B50C8` purple for focus states. Primary brand color is `#231212`. Reference implementations live in `packages/ui/src/`.
- No third-party component libraries (antd, shadcn, etc.) — use the custom components from `packages/ui`.

### Database

- Schema lives in `packages/db/src/schema.ts`. All tables shared across apps go here.
- App-specific tables (if ever needed) go in `apps/webapp/src/db/schema.ts`.
- Never call Drizzle directly from a page or component. Always go through a tRPC procedure or a server action.
- Migrations are generated with `pnpm db:generate` and applied with `pnpm db:migrate`.

### Authentication

- Use `getSession()` (from `@/lib/auth`) to get the session in RSC, middleware, and server actions.
- Use `protectedProcedure` for any tRPC endpoint that touches user data. This is the default — only use `publicProcedure` for explicit public endpoints.
- Protect pages at both the RSC level (`getSession()` → redirect) **and** in middleware — never rely on just one.
- Client-side auth state: use the Better Auth React hooks (`useSession`, `signIn`, `signOut`) from `@/lib/auth-client`.

### Logging

- Use the structured logger from `packages/logger` instead of `console.log`.
- Import via `import { logger } from "@travel-app/logger"`.
- Pass contextual data as the second argument: `logger.info("message", { userId })`.
- Log errors with `logger.error("message", { error: serializeError(err) })`.

### Error handling

- tRPC procedures: throw `TRPCError` with appropriate codes (`NOT_FOUND`, `UNAUTHORIZED`, `FORBIDDEN`, `BAD_REQUEST`, `INTERNAL_SERVER_ERROR`).
- Server actions: return `{ success: boolean; error?: string }` rather than throwing — callers should handle the error case.
- RSC: catch errors and render fallback UI; never let an unhandled error crash a page.
- Client mutations: wrap tRPC calls in try/catch, show errors via a toast or inline message.

### Dev sandbox

- `/components` — component library preview. **Dev only** (`NODE_ENV === 'development'`). Returns `notFound()` in production.

## Commands

```bash
pnpm dev              # start all apps (turbo)
pnpm build            # production build
pnpm lint             # run ESLint across all packages
pnpm format           # run Prettier across all files
pnpm format:check     # check formatting without writing
pnpm typecheck        # tsc --noEmit across all packages
pnpm test             # vitest run
pnpm test:coverage    # vitest with coverage report
pnpm db:generate      # generate drizzle migration
pnpm db:migrate       # apply migrations
pnpm db:push          # push schema changes directly (dev)
pnpm db:reset         # reset database
pnpm db:seed          # seed database
pnpm db:studio        # drizzle studio
pnpm i18n:generate    # generate i18n translation files
pnpm i18n:check       # check translation completeness
docker compose up -d  # start postgres
```

## What to avoid

- `any` types
- `useEffect` for data fetching — use RSC or tRPC queries
- Business logic in page components
- Huge components (>150 lines) — split them
- Adding new third-party UI libraries without discussion
- Skipping input validation on tRPC mutations
- Direct DB calls outside of tRPC routers

---
> Source: [ryanrawlingswang/llm-rules](https://github.com/ryanrawlingswang/llm-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
