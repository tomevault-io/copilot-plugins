## taru-villas

> Taru Villas is a **Next.js 16 survey management and quality assessment platform** for hotel property management. It features weighted scoring analytics, guest surveys, automatic task/issue tracking from low-score responses, and role-based access control.

# Taru Villas - Project Guide

## Overview

Taru Villas is a **Next.js 16 survey management and quality assessment platform** for hotel property management. It features weighted scoring analytics, guest surveys, automatic task/issue tracking from low-score responses, and role-based access control.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.1.6 (App Router, React 19) |
| Language | TypeScript 5 |
| Database | Supabase PostgreSQL via `postgres` (postgres.js) + Drizzle ORM |
| Auth | Supabase Auth (Google OAuth) |
| UI | shadcn/ui + Radix UI + Tailwind CSS 4 |
| Forms | React Hook Form + Zod v4 |
| Charts | Recharts |
| Tables | TanStack React Table |
| URL State | nuqs |
| Deployment | Vercel (serverless) |

## Project Structure

```
src/
├── app/
│   ├── (auth)/           # Login, OAuth callback (no auth required)
│   ├── (portal)/         # Authenticated routes
│   │   ├── admin/        # Admin-only pages (properties, users, templates, tasks)
│   │   ├── dashboard/    # Dashboard overview + property dashboards
│   │   ├── surveys/      # Survey list, creation, detail views
│   │   ├── tasks/        # Task list + detail (admin & PM)
│   │   └── settings/     # User settings
│   ├── (public)/         # Guest survey pages (token-based, no auth)
│   │   └── g/[token]/    # Guest survey form
│   └── api/              # API routes
│       ├── admin/guest-links/
│       ├── dashboard/
│       ├── properties/[id]/
│       ├── surveys/[id]/ + guest/
│       ├── tasks/[id]/
│       ├── templates/[id]/
│       └── users/[id]/
├── components/
│   ├── admin/            # Admin page client components
│   ├── dashboard/        # Dashboard overview + property charts
│   ├── layout/           # Sidebar, header
│   ├── surveys/          # Survey wizard, form, filters, score display
│   ├── tasks/            # Task list, detail
│   └── ui/               # shadcn/ui primitives
├── lib/
│   ├── auth/guards.ts    # requireAuth(), requireRole(), getProfile(), getUserProperties()
│   ├── db/
│   │   ├── index.ts      # Postgres connection (prepare: false for PgBouncer)
│   │   ├── schema.ts     # All tables, enums, relations
│   │   └── queries/      # Query functions by domain
│   │       ├── surveys.ts
│   │       ├── properties.ts
│   │       ├── profiles.ts
│   │       ├── tasks.ts
│   │       ├── guest-links.ts
│   │       └── dashboard.ts
│   ├── supabase/
│   │   ├── server.ts     # Server-side Supabase client
│   │   ├── admin.ts      # Service-role Supabase client
│   │   └── client.ts     # Browser Supabase client
│   └── utils.ts          # cn() helper (clsx + tailwind-merge)
├── middleware.ts          # Auth middleware (session check, public route exclusions)
drizzle/                   # Migration files
```

## Database Schema

### Enums
- `user_role`: admin | property_manager | staff
- `submission_status`: draft | submitted | reviewed
- `survey_type`: internal | guest
- `task_status`: open | investigating | closed

### Tables & Relationships

```
organizations (multi-tenant root)
├── profiles (auth users, FK to supabase auth.users)
│   └── propertyAssignments (M2M: user ↔ property)
├── properties
│   └── primaryPmId → profiles (default task assignee)
├── surveyTemplates
│   └── surveyCategories (weighted)
│       └── surveySubcategories
│           └── surveyQuestions (scale 1-10)
├── surveySubmissions
│   ├── surveyResponses (score + optional note + issueDescription)
│   └── guestSurveyLinks (token-based public access)
└── tasks (auto-created from low-score responses)
```

### Key Schema Details
- Survey templates have a **3-level hierarchy**: categories → subcategories → questions
- Categories have a `weight` field used for weighted average scoring
- Questions have configurable `scaleMin`/`scaleMax` (default 1-10)
- Submissions auto-generate slugs: `template-name-property-code-YYYY-MM-DD`
- Responses cascade-delete when submission is deleted
- Tasks are auto-created when response score <= 6 AND has issueDescription
- Tasks detect repeat issues (same question + property, previously closed)
- Guest links have unique constraint on (templateId, propertyId)

## Authentication & Authorization

### Auth Flow
1. Google OAuth via Supabase Auth
2. Auto-provisioning on first login (first user = admin, rest = staff)
3. Users must have @taruvillas.com email
4. Middleware checks session on every request

### Public Routes (no auth)
- `/login`, `/callback`
- `/g/*` (guest surveys)
- `/api/surveys/guest`
- `/api/cron/*` (uses Bearer CRON_SECRET)

### Role Permissions

| Route/Feature | admin | property_manager | staff |
|---------------|-------|-------------------|-------|
| `/dashboard` overview | Full access | Redirect to /surveys | Redirect to /surveys |
| `/dashboard/[propertyId]` | All properties | Assigned properties only | Redirect to /surveys |
| `/surveys` (list/create) | All | All | All |
| `/admin/*` pages | Full CRUD | No access | No access |
| `/tasks` | All org tasks | Assigned property tasks | No access (403) |
| Property/User CRUD APIs | Full | Read only | Read only |

### Auth Guards (`src/lib/auth/guards.ts`)
- `requireAuth()` — Redirects to /login, returns profile + assignments
- `requireRole(roles)` — Checks role, redirects to /surveys if unauthorized
- `getProfile()` — API-level, returns null (no redirect)
- `getUserProperties()` — Returns null for admins (= all access), property ID list for others

### Dev Bypass
Set `DEV_BYPASS_AUTH=true` to skip auth (returns mock admin profile).

## Key Business Logic

### Survey Scoring
- **Normalized score**: `((score - scaleMin) / (scaleMax - scaleMin)) * 10`
- **Weighted average**: `sum(normalized_score * category_weight) / sum(category_weight)`
- Dashboard queries in `src/lib/db/queries/dashboard.ts`

### Task Auto-Creation (`src/lib/db/queries/tasks.ts`)
1. Triggered on survey submission
2. For each response with score <= 6 AND issueDescription:
   - Creates task with title = question text, description = issueDescription
   - Assigns to property's primaryPmId (if set)
   - Checks for repeat issues (same question + property, previously closed)
3. Task lifecycle: open → investigating → closed (with closing notes)

### Guest Surveys
- Admin creates "guest" type template + generates guest link
- Link URL: `/g/[token]` (22-char base64url token)
- Guest fills survey without auth, records guestName/guestEmail
- Links can be toggled active/inactive

### Fixed Asset Registry
- Property-scoped asset registry with rooms, straight-line depreciation (computed on read), maintenance logs, QR labels + mobile scan, guard-layer RBAC (staff financial-blind); routes under `/assets`

## Critical Implementation Notes

### Database Connection
```typescript
// MUST use { prepare: false } — PgBouncer on Vercel breaks prepared statements
const client = postgres(connectionString, { prepare: false });
```

### Zod v4
Project uses Zod v4 (`^4.3.6`). Avoid strict `.url()` validators — use plain `.string()` for URL fields (query params can fail strict validation).

### Server Components
- Pages use React Server Components by default
- Data fetching via direct db query calls (no REST from server components)
- Dashboard pages use `export const dynamic = 'force-dynamic'` for fresh data

### Middleware
- `src/middleware.ts` runs on all non-static requests
- Handles Supabase session refresh
- Redirects unauthenticated → /login, authenticated on /login → /dashboard

## Environment Variables

```bash
# Required
NEXT_PUBLIC_SUPABASE_URL="https://..."
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJhbGc..."
NEXT_PUBLIC_APP_URL="https://tvpl.morpheusds.com"  # Base URL for asset QR links; must be set in Coolify as "Available at Buildtime"
SUPABASE_SERVICE_ROLE_KEY="eyJhbGc..."
POSTGRES_URL="postgres://user:pass@host:6543/db"  # Transaction mode (port 6543)

# Optional
DATABASE_URL="..."          # Fallback for POSTGRES_URL
DEV_BYPASS_AUTH="true"      # Skip auth in development
CRON_SECRET="..."           # Bearer token for /api/cron routes

# Oracle OHIP (Guest Profiles / pre-arrival) — sandbox values for now
ORACLE_OHIP_GATEWAY="https://<customer-gateway-host>"
ORACLE_OHIP_CLIENT_ID="..."
ORACLE_OHIP_CLIENT_SECRET="..."
ORACLE_OHIP_APP_KEY="..."
ORACLE_OHIP_USERNAME="..."
ORACLE_OHIP_PASSWORD="..."
```

## Development Commands

```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # ESLint
npx drizzle-kit generate   # Generate migration from schema changes
npx drizzle-kit migrate    # Run pending migrations
npx drizzle-kit studio     # Open Drizzle Studio (DB browser)
npx vercel deploy --prod --yes  # Deploy to Vercel
```

## Deployment

- **Platform**: Vercel
- **Team**: `team_93mQ4vskMbxT3b8AEH3IhXSL`
- **Project**: `prj_aCJKMwQJYr5Je2dmKagR8k9B8C1E`
- **Cron config**: `vercel.json`
- **Deploy**: `npx vercel deploy --prod --yes`

---
> Source: [sonaljay/Taru-Villas](https://github.com/sonaljay/Taru-Villas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
