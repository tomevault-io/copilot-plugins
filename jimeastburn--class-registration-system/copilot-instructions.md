## class-registration-system

> > **This is the #1 project rule. Violating TDD is NEVER acceptable. No exceptions. No shortcuts.**

# Project Guidelines

## Testing Methodology — MANDATORY TDD (Red/Green)

> [!CAUTION]
> **This is the #1 project rule. Violating TDD is NEVER acceptable. No exceptions. No shortcuts.**

### Hard Gates — Enforced at Every Stage

1. **PLANNING gate**: Every implementation plan MUST include a **"Tests (RED)"** section before any **"Implementation (GREEN)"** section. If the plan does not contain test code/descriptions, the plan is incomplete and MUST NOT be approved.

2. **EXECUTION gate**: During execution, test files MUST be created and run (failing) BEFORE writing any implementation code. The sequence is always:
   - Write the test → Run it → Confirm it FAILS (RED)
   - Write the minimal implementation → Run it → Confirm it PASSES (GREEN)
   - Never reverse this order. Never skip the RED step.

3. **VERIFICATION gate**: `npm run test:run` must be run at the end. New test count must be > 0 for any new feature or bug fix.

### Pre-Flight Checklist — STOP Before Every Code Change

> [!CAUTION]
> **Before writing or editing ANY implementation file, ask yourself these questions. If any answer is YES, you MUST write tests FIRST.**

1. Am I adding a new component, function, or helper? → **YES = tests first**
2. Am I changing what gets rendered or displayed? → **YES = tests first**
3. Am I adding conditional logic (if/else, filtering, mapping)? → **YES = tests first**
4. Am I fixing a bug? → **YES = write a test that reproduces the bug first**
5. Does this change affect data flow or user-visible behavior? → **YES = tests first**

**There is NO size exemption.** A "small" change, a "simple" UI tweak, or a "quick" helper function still requires TDD. The size of the change does not excuse skipping the process.

### What Requires Tests

- Every new server action
- Every new API route
- Every new utility/helper function
- Every new UI component (even small ones like pills, badges, indicators)
- Every new rendering helper or sub-component added to an existing component
- Every bug fix (write a test that reproduces the bug first)
- UI behavior changes that affect data flow (e.g., checkbox toggles that call server actions)
- Conditional rendering logic (e.g., showing/hiding elements based on props or state)

## Tech Stack

- **Framework**: Next.js 16 (App Router, Turbopack)
- **Language**: TypeScript, React 19.2.0
- **Styling**: Tailwind CSS v4 + shadcn/ui
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **Payments**: Stripe
- **Validation**: Zod

## Context7 Library IDs

Use these pre-resolved Library IDs when querying Context7 to ensure version compatibility. Run `resolve-library-id` if these are missing or outdated.

| Library          | Version         | Context7 ID                   |
| ---------------- | --------------- | ----------------------------- |
| **Next.js**      | 16 (App Router) | `/vercel/next.js`             |
| **React**        | 19              | `/facebook/react`             |
| **Tailwind CSS** | v4              | `/websites/tailwindcss`       |
| **Supabase**     | Latest          | `/supabase/supabase-js`       |
| **Stripe**       | Latest          | `/stripe/stripe-node`         |
| **Zod**          | v4              | `/colinhacks/zod`             |
| **Shadcn/UI**    | Latest          | `/shadcn-ui/ui`               |
| **Resend**       | Latest          | `/resend/resend-node`         |
| **Zoho Books**   | v3 API          | `/websites/zoho_books_api_v3` |

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Login, register pages
│   ├── (dashboard)/       # Role-based dashboards
│   │   ├── parent/        # Parent portal
│   │   ├── teacher/       # Teacher portal
│   │   ├── student/       # Student portal
│   │   ├── admin/         # Admin portal
│   │   └── class-scheduler/ # Class Scheduler portal
│   └── api/               # API routes (checkout, webhooks)
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── admin/             # Admin action components
│   ├── auth/              # Auth forms
│   ├── classes/           # Class & enrollment components
│   ├── dashboard/         # Shared dashboard layout
│   ├── family/            # Family member components
│   ├── payments/          # Payment components
│   └── class-scheduler/   # Class Scheduler components
├── lib/
│   ├── supabase/          # Supabase client config
│   ├── actions/           # Server actions
│   └── validations.ts     # Zod schemas
├── hooks/                 # Custom React hooks
└── types/                 # TypeScript types
```

## Documentation

- [System Requirements](./docs/REGISTRATION_SYSTEM_DESCRIPTION.md)
- [Architecture Decisions](./docs/architecture_decision_document.md)
- [API Specification](./docs/api_planning_document.md)
- [Zoho Integration Flow](./docs/zoho_integration_flow.md)
- [Deployment Guide](./docs/DEPLOYMENT.md)
- [Testing Guide](./docs/TESTING.md)
- [Manual Testing Guide](./docs/MANUAL_TESTING.md)
- [Design System](./docs/DESIGN_SYSTEM.md)
- [Task List](./docs/TASKS.md)

> [!NOTE]
> Skills are available in the `.agent/skills` directory.

## Development Commands

```bash
npm run dev    # Start dev server (Turbopack)
npm run build  # Production build
npm run lint   # Run ESLint
npm run test:run   # Run all tests (non-interactive)
supabase gen types --lang=typescript --project-id jakjpigeafqqgispwlhl --schema public > src/types/database.ts  # Regenerate DB types
```

## Environment Variables

Copy `.env.example` to `.env.local` and configure:

```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=       # For webhooks

# Stripe
STRIPE_SECRET_KEY=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
STRIPE_WEBHOOK_SECRET=

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## User Roles

| Role      | Access                                                      |
| --------- | ----------------------------------------------------------- |
| `parent`  | Manage family, enroll children, pay fees                    |
| `teacher` | Create/manage classes, view students, AND manage own family |
| `student` | View schedule and class details                             |
| `admin`   | Full system access, data exports, AND manage own family     |

## Safety & Integrity Constraints

The following business logic safeguards are strictly enforced:

- **Webhook Idempotency**: Stripe `checkout.session.completed` events are checked against the database. Duplicate webhooks skip side-effects (Sync/Email) to prevent redundant billing.
- **Fault Tolerance**: Zoho sync failures do not block enrollment confirmation. Data is preserved for later manual or automated retry.
- **Capacity Hand-off**: Class capacity is atomically checked. Vacated spots (cancellations/deletions) correctly release capacity for the waitlist.
- **CSV Hardening**: All administrative data exports are escaped using `'` to prevent spreadsheet formula injection.
- **Privilege Revocation**: Role demotions (Admin/Teacher/Class Scheduler -> Parent) immediately revoke all elevated action access.
- **Cross-Cutting Refactors**: Any property rename, type change, or column rename MUST follow the `.agent/skills/systematic-refactoring/SKILL.md` audit process before editing files. No exceptions.
- **Migration File Sync**: Every call to `mcp_supabase_apply_migration` MUST be immediately followed by creating a matching local SQL file in `supabase/migrations/<version>_<name>.sql` with identical content. No exceptions—remote and local migrations must always stay in sync.
- **Dual-Database Migrations**: Every migration MUST be pushed to BOTH Supabase databases: **production** (`jakjpigeafqqgispwlhl`) and **dev/preview** (`nztngdpneuyhhnrkhehq`). After pushing, run `supabase migration list` against both to verify Local and Remote are fully aligned. Always re-link back to **production** when done. Use the `/push-migrations` workflow for the full procedure.

## Type Regeneration After Migrations — MANDATORY

> [!CAUTION]
> **Stale types cause silent drift between the schema and application code. Never skip this. No shortcuts.**

### Hard Gates — Enforced After Every Migration

1. **EXECUTION gate**: After any database migration is applied (locally or remotely), you MUST follow the `/post-migration` workflow **before** writing any application code that depends on the new schema. Never skip this. Never substitute a grep for the actual regeneration.

2. **VERIFICATION gate**: `npm run build` must pass with zero type errors after regeneration. If it fails, fix the type errors before proceeding.

### What Triggers This

- Every `mcp_supabase_apply_migration` call
- Every `supabase db push` or `supabase migration up`
- Every manual SQL change to the database schema
- Any time you are asked "do the types need updating?" (run the workflow; do NOT grep)
- **Type Reuse**: Before defining new interfaces for Supabase data shapes, ALWAYS check `src/types/database.ts` (generated) and `src/types/index.ts` (project-level) for existing types. Prefer importing and extending these over hand-rolling new interfaces. For joined queries, compose types using `Pick<>` from existing interfaces (e.g., `Pick<Class, 'name' | 'location' | 'block' | 'schedule_config'>`). Only create local types if no suitable base exists.

---
> Source: [JimEastburn/class-registration-system](https://github.com/JimEastburn/class-registration-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
