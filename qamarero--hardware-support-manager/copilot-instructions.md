## hardware-support-manager

> This file provides guidance to Codex when working with this repository.

# AGENTS.md

This file provides guidance to Codex when working with this repository.

## Project Overview

Hardware Support Manager is an internal web application for a hardware support department that acts as intermediary between clients, providers, and warehouse/office. It manages **incidents** (support tickets) and **RMAs** (Return Merchandise Authorizations) through their full lifecycle using state machines, with complete audit trails, aging tracking, and file attachments.

**Target users**: Internal support team members (not end clients).
**Language**: All UI labels, states, form fields, error messages, and user-facing text must be in **Spanish**.

## Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript (strict mode) |
| ORM | Drizzle ORM |
| Database | Neon PostgreSQL (serverless driver) |
| UI Components | shadcn/ui + Tailwind CSS v4 |
| Server State | TanStack Query v5 |
| URL State | nuqs |
| Forms | React Hook Form + Zod |
| Authentication | NextAuth.js v5 (Auth.js) with credentials provider |
| File Storage | Vercel Blob (abstracted behind storage layer) |
| Charts | Recharts |
| Notifications | Sonner (toast) |
| Testing | Vitest |

## Essential Commands

```bash
npm run dev          # Start development server (localhost:3000)
npm run build        # Production build (also validates types)
npm run lint         # Run ESLint
npm run db:push      # Push Drizzle schema directly to Neon (dev only)
npm run db:migrate   # Run generated Drizzle migrations (production)
npm run db:generate  # Generate migration files from schema changes
npm run db:seed      # Seed database with demo data
npm run db:studio    # Open Drizzle Studio (database GUI)
npm test             # Run Vitest tests
npm run test:watch   # Vitest in watch mode
npm run test:coverage # Vitest with coverage report
```

## Project Structure

```
src/
  app/                          # Next.js App Router
    (auth)/                     # Auth pages (login)
    (dashboard)/                # Authenticated layout group
      dashboard/                # Dashboard/home page
      incidents/                # Incident pages (list, detail, create, edit)
      rmas/                     # RMA pages (list, detail, create, edit)
      clients/                  # Client management
      providers/                # Provider management
      users/                    # User management (admin)
      settings/                 # App settings
    api/                        # API routes (only /api/upload for file uploads)
    layout.tsx                  # Root layout
  components/
    ui/                         # shadcn/ui base components
    layout/                     # App shell: sidebar, header, breadcrumbs
    incidents/                  # Incident-specific components (forms, tables, detail views)
    rmas/                       # RMA-specific components
    dashboard/                  # Dashboard widgets and charts
    shared/                     # Reusable components (data-table, file-uploader, state-badge, etc.)
  lib/
    db/
      index.ts                  # Drizzle client (Neon serverless)
      schema/                   # Drizzle table definitions (one file per entity)
      migrations/               # Migration utilities
    auth/                       # NextAuth.js v5 configuration
    storage/                    # File storage abstraction (Vercel Blob)
    validators/                 # Zod schemas (shared between client and server)
    state-machines/             # Incident and RMA state machine definitions
    utils/                      # Helper functions (formatting, dates, ID generation)
    constants/                  # App-wide constants (states, roles, categories)
  server/
    actions/                    # Server Actions (mutations)
    queries/                    # Server-side data fetching functions
  hooks/                        # Custom React hooks
  types/                        # TypeScript type definitions and interfaces
drizzle/                        # Generated migration SQL files
public/                         # Static assets (images, icons)
docs/                           # Project documentation
```

## Code Conventions

### Naming

| Element | Convention | Example |
|---|---|---|
| Files & folders | kebab-case | `incident-form.tsx`, `state-machines/` |
| Functions & variables | camelCase | `getIncidentById`, `isLoading` |
| React components | PascalCase | `IncidentDetail`, `RmaForm` |
| Constants | UPPER_SNAKE_CASE | `INCIDENT_STATES`, `MAX_FILE_SIZE` |
| Database tables | snake_case | `incidents`, `event_logs` |
| Database columns | snake_case | `created_at`, `client_id` |
| TypeScript types | PascalCase | `Incident`, `RmaStatus` |
| Zod schemas | camelCase with Schema suffix | `createIncidentSchema`, `updateRmaSchema` |

### File Organization

- One component per file. File name matches component name in kebab-case.
- Co-locate component-specific types and utilities with the component.
- Shared types go in `src/types/`.
- All Zod validators in `src/lib/validators/` (shared between client forms and server actions).

### API Pattern: Server Actions

All data mutations use **Server Actions** (no REST API endpoints). The only exception is `/api/upload` for file uploads (multipart form data).

```typescript
// src/server/actions/incidents.ts
"use server";

import { createIncidentSchema } from "@/lib/validators/incident";
import { db } from "@/lib/db";

export async function createIncident(formData: FormData) {
  const parsed = createIncidentSchema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.flatten() };
  // ... insert into DB, return result
}
```

### Server State with TanStack Query

Use TanStack Query v5 for all client-side data fetching and cache management. Server Actions are called via `useMutation`. Data fetching functions from `src/server/queries/` are used with `useQuery`.

### URL State with nuqs

Use nuqs for all URL-synchronized state (filters, pagination, search, sorting, tabs). This keeps UI state shareable and bookmarkable.

### Forms

All forms use React Hook Form with Zod resolver. Validators are defined once in `src/lib/validators/` and shared between client validation and server-side parsing.

### Paths

- Always use **relative** imports with the `@/` alias (mapped to `src/`).
- Never hardcode absolute filesystem paths.
- Use `path.join()` when building paths programmatically.

## Domain Model

### Incidents (INC-YYYY-NNNNN)

Support tickets tracking hardware issues from report to resolution.

- **States**: Defined by state machine in `src/lib/state-machines/incident.ts`
- **Key fields**: client, device info, category, priority, description, assigned user
- **Features**: State transitions with validation, aging tracking (time in current state), event log (audit trail), file attachments

### RMAs (RMA-YYYY-NNNNN)

Return Merchandise Authorizations for sending defective hardware to providers.

- **States**: Defined by state machine in `src/lib/state-machines/rma.ts`
- **Key fields**: linked incident(s), provider, device info, tracking numbers
- **Features**: State transitions, provider communication tracking, device location tracking, event log, file attachments

### Supporting Entities

- **Clients**: Companies or individuals reporting issues
- **Providers**: Hardware manufacturers/distributors for RMA processing
- **Users**: Internal team members with roles (admin, technician, viewer)
- **EventLog**: Polymorphic audit trail (linked to incidents or RMAs)
- **Attachments**: Polymorphic file attachments (linked to incidents, RMAs, or event log entries)

### ID Format

Both incidents and RMAs use the format `{PREFIX}-{YEAR}-{SEQUENTIAL_5_DIGITS}`:
- `INC-2026-00001`, `INC-2026-00002`, ...
- `RMA-2026-00001`, `RMA-2026-00002`, ...

Sequential counter resets each year.

## Database

### Drizzle ORM with Neon

- Schema files in `src/lib/db/schema/` (one file per table/entity).
- Client initialization in `src/lib/db/index.ts` using Neon serverless driver.
- Use `drizzle-kit` commands for migrations (see Essential Commands).

### Schema Changes Workflow

1. Modify schema files in `src/lib/db/schema/`.
2. Run `npm run db:generate` to create migration SQL.
3. Run `npm run db:migrate` to apply (or `npm run db:push` in development).
4. Update corresponding Zod validators if fields changed.
5. Update TypeScript types if needed.

### Conventions

- Use `pgTable` from `drizzle-orm/pg-core`.
- All tables include `id` (UUID, primary key), `createdAt`, and `updatedAt` timestamps.
- Use database-level foreign keys and constraints.
- Soft deletes where appropriate (`deletedAt` nullable timestamp).

## Authentication

NextAuth.js v5 (Auth.js) with credentials provider.

- Configuration in `src/lib/auth/`.
- Middleware protects all routes under `(dashboard)/`.
- Auth pages under `(auth)/` layout group.
- Session available via `auth()` on server and `useSession()` on client.
- Roles: `admin`, `technician`, `viewer` -- enforce in server actions.

## File Storage

Abstracted storage layer in `src/lib/storage/`:

- **Upload endpoint**: `/api/upload` (the only REST endpoint).
- **Storage backend**: Vercel Blob.
- **Abstraction**: Storage functions are behind an interface so the backend can be swapped.
- **Attachments are polymorphic**: Can be linked to incidents, RMAs, or event log entries.
- **Constraints**: Validate file type and size before upload.

## Testing

```bash
npm test              # Run all tests
npm run test:watch    # Watch mode
npm run test:coverage # Coverage report
```

### Priorities

1. **State machines**: Test all valid transitions and reject invalid ones.
2. **Validators**: Test Zod schemas with valid and invalid data.
3. **Utils**: Test ID generation, date formatting, aging calculations.
4. **Server actions**: Test with mocked DB where practical.

### Conventions

- Test files live next to source files: `incident.test.ts` beside `incident.ts`.
- Use Vitest `describe`/`it` blocks with descriptive names (in English).
- Mock external services (DB, storage) in unit tests.

## Deployment

- **Platform**: Vercel
- **Database**: Neon PostgreSQL (connection via serverless driver)
- **File storage**: Vercel Blob
- **Environment variables**: Set in Vercel dashboard (never in code)

### Deploy Workflow

```bash
npm run build         # Verify build passes locally
npm run lint          # Verify no lint errors
npm test              # Verify tests pass
# Push to main branch -- Vercel deploys automatically
```

## Security

### Environment Variables

- **NEVER** hardcode secrets, API keys, tokens, or passwords in code.
- Use `process.env.VARIABLE_NAME` to access secrets.
- Local development: `.env` or `.env.local` (already in `.gitignore`).
- Production: Vercel environment variables dashboard.

```typescript
// WRONG
const dbUrl = "postgresql://user:password@host/db";

// CORRECT
const dbUrl = process.env.DATABASE_URL;
```

### Required Environment Variables

```bash
DATABASE_URL=              # Neon PostgreSQL connection string
NEXTAUTH_SECRET=           # NextAuth.js secret (generate with openssl rand -base64 32)
NEXTAUTH_URL=              # App URL (http://localhost:3000 in dev)
BLOB_READ_WRITE_TOKEN=     # Vercel Blob token
```

### Auth Security

- All server actions must verify session before executing.
- Role-based access: check user role for destructive or admin-only operations.
- Validate all inputs with Zod on the server side, even if also validated on client.

## Common Issues

**Build fails with type errors**
- Run `npm run build` locally before pushing. Next.js build is stricter than `tsc`.
- Check that all server actions have `"use server"` directive.

**Database connection errors**
- Verify `DATABASE_URL` is set and correct.
- Neon databases may sleep after inactivity -- first request may be slow.
- Ensure using the Neon serverless driver (`@neondatabase/serverless`), not `pg`.

**Drizzle schema out of sync**
- Run `npm run db:push` (dev) or `npm run db:migrate` (prod) after schema changes.
- If `db:push` fails, check for breaking changes (dropping columns with data).

**File uploads failing**
- Verify `BLOB_READ_WRITE_TOKEN` is set.
- Check file size limits in the upload route.
- Ensure the `/api/upload` route handles multipart form data correctly.

**State transition rejected**
- Check the state machine definition for allowed transitions.
- Verify the current state is correct (may be stale -- refetch).
- Check if the transition requires specific conditions (e.g., all fields filled).

**Spanish characters not displaying**
- Ensure files are saved as UTF-8.
- Check that `<html lang="es">` is set in the root layout.

**TanStack Query cache stale after mutation**
- Invalidate relevant query keys after successful server action mutations.
- Use `queryClient.invalidateQueries({ queryKey: [...] })` in `onSuccess`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qamarero)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qamarero)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
