## elevare-saas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Elevare-SaaS is a project and task management SaaS application built with Next.js 14 (App Router), TypeScript, Prisma 6, PostgreSQL (Supabase), and NextAuth.js.

## Commands

```bash
npm run dev          # Start development server
npm run build        # Generate Prisma client + Next.js production build
npm run start        # Start production server
npm run lint         # Run ESLint
npm run db:seed      # Seed demo data (user: john@example.com / password)
```

Prisma commands:
```bash
npx prisma generate  # Regenerate Prisma client after schema changes
npx prisma db push   # Push schema changes to database
npx prisma studio    # Open database GUI
```

No test framework is configured.

## Architecture

### Route Groups

The app uses Next.js App Router with two route groups under `src/app/`:

- **`(auth)/`** — Login and register pages (public)
- **`(main)/`** — Dashboard, projects, tasks, settings (protected by middleware)

Route protection is handled in `middleware.ts` using NextAuth's `withAuth`. Protected paths: `/dashboard`, `/projects/*`, `/tasks/*`, `/settings/*`.

### Auth Flow

- **Config**: `src/lib/auth.ts` — CredentialsProvider with bcryptjs, JWT sessions (24h), PrismaAdapter
- **Session**: `getServerSession(authOptions)` in API routes to get the authenticated user
- **Registration**: `POST /api/register` is the only public API endpoint (no auth required)

### Data Layer

- **Prisma schema**: `prisma/schema.prisma` — 5 models: User, Project, Task, Activity, Comment
- **Prisma client singleton**: `src/lib/prisma.ts`
- **All data is user-scoped** — API routes filter by `userId` from the session
- **Cascade deletes**: Deleting a user cascades to all their projects, tasks, activities, comments

### API Pattern

API routes live in `src/app/api/`. Each route:
1. Gets session via `getServerSession(authOptions)`
2. Returns 401 if no session
3. Performs Prisma operations scoped to `session.user.id`
4. Logs actions via `activityLog()` from `src/lib/utils.ts`

### Client-Side Pattern

- **Services** (`src/services/`): Fetch wrappers for each API resource (projects, tasks, activities, users)
- **Hooks** (`src/hooks/`): Custom React hooks that use services and manage state (useProjects, useTask, useComments, etc.)
- **Components** consume hooks — they don't call APIs directly

### Key Utilities (`src/lib/utils.ts`)

- `cn()` — Tailwind class merging (clsx + tailwind-merge)
- `activityLog()` — Creates activity records in the database
- `formatDate()` / `formatDateTime()` — Date formatting helpers
- `getStatusColor()` / `getPriorityColor()` — Status/priority to color mapping

### UI Components

Uses shadcn/ui pattern (New York style) with Radix UI primitives. Components live in `src/components/ui/`. Config in `components.json`. Import alias: `@/components/ui/`.

### Path Alias

`@/*` maps to `src/*` (configured in tsconfig.json).

## Environment Variables

Required in `.env`:
- `DATABASE_URL` — PostgreSQL connection string (Supabase)
- `NEXTAUTH_SECRET` — JWT signing secret

## Conventions

- Status values are strings: `"active"`, `"planning"`, `"archived"` (projects); `"todo"`, `"in_progress"`, `"done"` (tasks)
- Priority values are strings: `"low"`, `"medium"`, `"high"`
- Task ordering uses `position` (integer) field; kanban uses `kanbanPosition`
- Client components are marked with `"use client"`
- Prettier: 2-space indent, no tabs
- Build config ignores ESLint and TypeScript errors (`next.config.mjs`)

## Behavior Rules
- **Always ask before making important decisions** — do not assume or decide 
  unilaterally on anything that affects architecture, data models, file structure, 
  or business logic. When in doubt, ask the user first.

---
> Source: [alan-ari-mahendra/Elevare-SaaS](https://github.com/alan-ari-mahendra/Elevare-SaaS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
