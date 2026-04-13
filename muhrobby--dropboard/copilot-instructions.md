## dropboard

> Full-stack Next.js 16 (App Router) asset management platform. TypeScript strict mode, Tailwind CSS v4, Drizzle ORM + PostgreSQL, TanStack Query, Zustand, better-auth, shadcn/ui.

# AGENTS.md — Dropboard Coding Guidelines

## Project Overview

Full-stack Next.js 16 (App Router) asset management platform. TypeScript strict mode, Tailwind CSS v4, Drizzle ORM + PostgreSQL, TanStack Query, Zustand, better-auth, shadcn/ui.

---

## Commands

```bash
# Dev server (port 3004, not 3000)
pnpm dev

# Production build
pnpm build

# Lint (ESLint 9 flat config)
pnpm lint

# Type-check (no emit)
npx tsc --noEmit

# Run all tests
pnpm test                          # vitest run (CI mode, no watch)
pnpm test:watch                    # vitest (watch mode)

# Run a single test file
pnpm test -- __tests__/item-service.test.ts

# Run tests matching a name pattern
pnpm test -- --reporter=verbose -t "should create item"

# Database
pnpm db:push        # sync schema changes to DB (after schema edits)
pnpm db:generate    # generate migration files
pnpm db:migrate     # apply migrations
pnpm db:studio      # open Drizzle Studio
pnpm db:seed        # seed the database
```

### Package Manager

Use **pnpm** exclusively (`pnpm-lock.yaml` present).

```bash
pnpm add <pkg>       # runtime dependency
pnpm add -D <pkg>    # dev dependency
```

---

## Tech Stack

| Layer | Library |
|---|---|
| Framework | Next.js 16, App Router, React 19 |
| Language | TypeScript 5, strict mode |
| Styling | Tailwind CSS v4, `@import "tailwindcss"` in globals.css |
| UI components | shadcn/ui (in `components/ui/`) |
| Fonts | `Inter` (sans) + `JetBrains_Mono` (mono) via `next/font/google` |
| Server state | TanStack Query v5 |
| Client state | Zustand v5 |
| ORM | Drizzle ORM + `postgres` driver |
| Database | PostgreSQL, ULID primary keys |
| Auth | better-auth, session cookies |
| Validation | Zod v4 — always import from `"zod/v4"` |
| Testing | Vitest 4 (integration, real DB, no mocks) |

---

## Project Structure

```
app/                  # Next.js App Router
  (auth)/             # Login, register, password reset
  api/v1/             # All REST API routes
  dashboard/          # Main app pages
  actions/            # Next.js server actions
components/
  drops/              # Drop-specific components
  shared/             # Generic reusable components
  todos/              # Todo/task components
  ui/                 # shadcn/ui primitives (do not edit)
  layout/             # Sidebar, topbar, nav
db/
  schema/             # Drizzle table definitions + index.ts barrel
  index.ts            # db instance (drizzle + postgres)
hooks/                # TanStack Query hooks (use-items.ts pattern)
lib/
  api-helpers.ts      # Typed NextResponse helpers
  errors.ts           # AppError hierarchy
  validations/        # Zod schemas + inferred types
  constants.ts        # App-wide constants + RBAC permission maps
  tier-guard.ts       # Subscription/quota checks
  file-storage.ts     # Disk storage + signed URLs
middleware/           # auth-guard.ts, workspace-guard.ts, rbac.ts
services/             # Business logic (item-service.ts pattern)
stores/               # Zustand stores (workspace-store, ui-store)
types/
  api.ts              # API response types (ApiResponse<T>, ItemResponse, etc.)
  index.ts            # Domain enums (ItemType, MemberRole, etc.)
__tests__/            # All Vitest tests live here
```

---

## Testing

Tests use **Vitest** against a **real PostgreSQL database** — no mocks, sequential execution.

- All test files go in `__tests__/` (the vitest config only picks up `__tests__/**/*.test.ts`)
- Setup file: `__tests__/setup.ts` loads `.env.local.test`
- Test DB: configure `DATABASE_URL` in `.env.local.test`
- Baseline: **103 passed / 1 skipped** — all must pass before merging
- `fileParallelism: false` — tests run sequentially, do not add parallelism

```bash
# Single file
pnpm test -- __tests__/item-service.test.ts

# Single test by name
pnpm test -- -t "createItem"
```

---

## Code Style

### TypeScript

- Strict mode is on — no `any`, no non-null assertions (`!`) without justification
- Prefer `type` over `interface` for object shapes
- Use inferred types from Zod schemas: `type CreateDropInput = z.infer<typeof createDropSchema>`
- Use `React.ReactNode` for children props
- Always annotate async function return types explicitly

### Imports

- Use `@/*` alias (maps to repo root): `import { db } from "@/db"`
- Import order: React/Next → third-party → local `@/` imports
- Use `import type` for type-only imports

```typescript
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod/v4";
import { db } from "@/db";
import type { ItemResponse } from "@/types/api";
```

### Zod — always use `"zod/v4"`

```typescript
import { z } from "zod/v4";   // correct
import { z } from "zod";       // wrong — will break at runtime
```

### Naming

| Thing | Convention |
|---|---|
| Components | PascalCase (`DropCard.tsx`) |
| Utility files | kebab-case (`file-storage.ts`) |
| Variables / functions | camelCase |
| Constants | UPPER_SNAKE_CASE |
| DB column names | snake_case (Drizzle maps to camelCase via `.$defaultFn`) |

### Components

- `"use client"` at top of any component using hooks, event handlers, or browser APIs
- Default export for pages and route segments; named exports for shared components
- Inline prop types for simple shapes; named `type Props = {...}` for complex ones
- **Never use `window.alert` or `window.confirm`** — use `<ConfirmDialog>` from `components/shared/confirm-dialog.tsx`
- Toast feedback via `sonner`: `import { toast } from "sonner"`

---

## API Routes

Every API route handler follows this pattern:

```typescript
export async function GET(request: NextRequest) {
  try {
    const session = await requireAuth();                           // throws UnauthorizedError
    const { workspaceId } = parseSearchParams(request);
    const parsed = myQuerySchema.safeParse({ workspaceId });
    if (!parsed.success) return validationErrorResponse("...");
    await requireWorkspaceMembership(session.user.id, workspaceId); // throws ForbiddenError
    const data = await myService(parsed.data);
    return successResponse(data);
  } catch (error) {
    if (error instanceof ForbiddenError) return forbiddenResponse(error.message);
    if (error instanceof NotFoundError) return notFoundResponse(error.message);
    if (error instanceof AppError) return unauthorizedResponse(error.message);
    return serverErrorResponse();
  }
}
```

- Import response helpers from `@/lib/api-helpers`
- Import error classes from `@/lib/errors`
- Import guards from `@/middleware/auth-guard` and `@/middleware/workspace-guard`
- Validate all inputs with Zod `safeParse` — never trust raw request data
- After DB schema changes, run `pnpm db:push`

---

## Error Handling

Custom error hierarchy in `lib/errors.ts`:

```
AppError (base)
├── NotFoundError       → 404
├── UnauthorizedError   → 401
├── ForbiddenError      → 403
├── ValidationError     → 422
└── QuotaExceededError  → 413
```

- Throw typed errors in services; catch and map them in route handlers
- Never expose internal error details in production — `lib/error-sanitizer.ts` handles this
- Always prefer early returns over nested conditionals

---

## Hooks (TanStack Query)

Follow the pattern in `hooks/use-items.ts`:

- Fetch functions are plain `async` functions defined above the hook (not inline)
- Query key: `["resource", activeWorkspaceId, ...params]`
- Enable query only when workspace is active: `enabled: !!activeWorkspaceId`
- Get workspace ID from Zustand: `const { activeWorkspaceId } = useWorkspaceStore()`
- Invalidate `["items"]` (or relevant key) on successful mutations
- Optimistic updates: implement `onMutate` + `onError` rollback for create/delete

---

## Database

- ORM: Drizzle with `postgres` driver — `import { db } from "@/db"`
- Primary keys: ULID via `ulid()` package, assigned with `.$defaultFn(() => ulid())`
- All schemas defined in `db/schema/`; barrel-exported from `db/schema/index.ts`
- Soft delete pattern: `deletedAt timestamp` column (not physical delete)
- Always add indexes for columns used in `WHERE` clauses (see `items.ts` for examples)
- After any schema edit: `pnpm db:push` (dev) or `pnpm db:generate && pnpm db:migrate` (prod)

---

## Stores (Zustand)

- `useWorkspaceStore` — persisted; provides `activeWorkspaceId`, `setActiveWorkspaceId`
- `useUIStore` — ephemeral; provides `isUploadModalOpen`, `isSidebarOpen` and their setters
- Do not add new stores without good reason; prefer TanStack Query for server state

---

## Git

- Do not commit: `.env*`, `node_modules/`, `.next/`, `out/`, `build/`, `uploads/`, `docs/`
- Follow conventional commits: `feat:`, `fix:`, `chore:`, `refactor:`, `test:`
- Run `npx tsc --noEmit` and `pnpm lint` before committing
- All 103 Vitest tests must pass: `pnpm test`

---

## Environment

- Dev port: **3004** (not 3000)
- Copy `.env.example` → `.env` and fill in values
- Test environment: `.env.local.test` (loaded by `__tests__/setup.ts`)
- Required vars: `DATABASE_URL`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL`, `NEXT_PUBLIC_APP_URL`, `SIGNED_URL_SECRET`, `CRON_SECRET`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhrobby) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
