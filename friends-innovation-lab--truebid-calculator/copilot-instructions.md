## truebid-calculator

> TrueBid helps government contractors build cost proposals. Users upload RFPs, the AI extracts requirements and generates Work Breakdown Structures (WBS), and the tool produces compliant cost estimates with BOE (Basis of Estimate) documentation. Exports to PDF/DOCX.

# TrueBid — AI-Powered Government Contracting Proposal Tool

## Project Overview

TrueBid helps government contractors build cost proposals. Users upload RFPs, the AI extracts requirements and generates Work Breakdown Structures (WBS), and the tool produces compliant cost estimates with BOE (Basis of Estimate) documentation. Exports to PDF/DOCX.

## Tech Stack

- **Framework:** Next.js 15 (App Router) with React 19
- **Language:** TypeScript 5 (strict mode)
- **Styling:** Tailwind CSS 4 + shadcn/ui (Radix primitives)
- **State:** React Context (`contexts/app-context.tsx` is the main state store)
- **Auth:** Supabase Auth with SSR middleware
- **Database:** Supabase (PostgreSQL with RLS)
- **AI:** Anthropic Claude API (`@anthropic-ai/sdk`) for RFP extraction and WBS generation
- **Validation:** Zod (schemas in `lib/schemas/`)
- **Testing:** Jest + React Testing Library (unit), Playwright (E2E)
- **Git Hooks:** Husky + lint-staged (ESLint on pre-commit)

## Project Structure

```
app/
  (auth)/              # Login, signup, forgot/reset password
  [id]/                # Dynamic proposal detail page
  account/             # User settings, company, billing, labor, rates, team
  api/
    companies/         # Company CRUD + roles + settings
    extract-rfp/       # AI: PDF → structured solicitation data
    generate-wbs/      # AI: requirements → WBS tasks
    proposals/         # Proposal CRUD + requirements + WBS
    user/              # Profile + avatar
  dashboard/           # Main dashboard
  onboarding/          # New user onboarding
  tools/               # Standalone tools page

components/
  account/             # Account settings components
  auth/                # Auth layout
  dashboard/           # Dashboard component
  shared/              # App header, footer, tabs navigation, layout wrapper
  tabs/                # Main proposal tabs (upload, estimate, export, etc.)
  ui/                  # shadcn/ui primitives (button, card, dialog, etc.)

contexts/
  app-context.tsx      # Central state: proposals, roles, settings, WBS, solicitation
  auth-context.tsx     # Auth state

lib/
  api.ts               # Client-side API helpers
  schemas/             # Zod validation schemas
  supabase/            # Supabase client (browser) + server (service role)
  utils.ts             # cn() utility
  export-utils.ts      # PDF/DOCX generation
  boe-export.ts        # BOE document export
  solicitation-type.ts # Solicitation/RFP type definitions
  gsa-schedule-data.ts # GSA schedule reference data

hooks/
  use-proposal-sync.ts # Proposal sync hook
```

## Commands

```bash
npm run dev           # Start dev server (localhost:3000)
npm run build         # Production build
npm run lint          # ESLint
npm run typecheck     # TypeScript check (tsc --noEmit)
npm test              # Run Jest unit tests
npm run test:watch    # Jest in watch mode
npm run test:coverage # Jest with coverage report
npm run test:e2e      # Playwright E2E tests
npm run test:e2e:ui   # Playwright with UI
```

## CRITICAL RULES

1. **ONE change at a time** — Never make multiple unrelated changes in a single session
2. **NO clever solutions** — Make the minimal change that fixes the issue
3. **NO React Portals, NO major refactors** unless explicitly asked
4. **ASK before committing** — Show me the diff first with `git diff`
5. **TEST before declaring done** — Verify the full flow works

## When Making Changes

1. Show me what you plan to change BEFORE editing
2. Make the smallest possible fix
3. Run `git diff` and show me the output
4. Wait for my approval before committing

## DO NOT

- Add new dependencies without asking
- Refactor working code
- Make "improvements" I didn't ask for
- Create multiple commits for one fix
- Use `@ts-nocheck` in new files
- Use `localStorage` for new features (use Supabase)

## Git Workflow

- Work on `develop` branch only unless explicitly told otherwise
- Do NOT create new branches without explicit permission
- Commit after each logical fix with clear messages (e.g., `fix: dashboard proposal title display`)
- Test changes before committing
- Pre-commit hook runs ESLint via lint-staged

## Database Schema

The Supabase `proposals` table has these columns:
- id, company_id, title, solicitation_number
- client, agency, client_agency (use client + agency, not client_agency)
- status, contract_type, due_date
- estimated_value, total_value, period_of_performance (jsonb)
- team_size, progress, starred, archived
- created_at, updated_at

## API Routes

- All routes use the authenticated user's session via `lib/supabase/server.ts` (anon key + cookies)
- RLS policies enforce data isolation — users can only access their own company's data
- Auth check: `supabase.auth.getUser()` at start of each route
- Return 401 if no session
- Log errors with full details before returning 500
- Validate request bodies with Zod schemas from `lib/schemas/`

## UI Standards

Before writing any UI code, read `DESIGN-SYSTEM.md`. All components, patterns, and decisions must follow it.

## Code Style

- snake_case for database columns
- camelCase for TypeScript/React
- Transform between them in API routes
- Use shadcn/ui components from `components/ui/`
- Follow existing patterns in the codebase

## Known Tech Debt

Files with `@ts-nocheck` (need gradual type fixing):
- `components/task-decomposition.tsx`
- `components/tabs/upload-tab.tsx`
- `components/tabs/export-tab.tsx`
- `components/tabs/rate-justification-tab.tsx`
- `components/tabs/sub-rates-tab.tsx`
- `components/tabs/roles-and-pricing-tab.tsx`

TypeScript strictness: `noUnusedLocals` and `noUnusedParameters` are not enabled yet due to many existing violations. Enable them incrementally.

ESLint: `@typescript-eslint/no-explicit-any` is currently disabled. Tighten once `@ts-nocheck` files are fixed.

## Environment Variables

See `.env.example` for required keys:
- `NEXT_PUBLIC_SUPABASE_URL` — Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` — Supabase anon/public key
- `SUPABASE_SERVICE_ROLE_KEY` — Supabase service role key (server only)
- `ANTHROPIC_API_KEY` — Anthropic Claude API key (server only)

## AI Features

- **Shipley PDF:** Available via `SHIPLEY_PDF_URL` env var. Used as knowledge base for the Phase 4 coaching engine. Never commit the actual URL to the repository.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/friends-innovation-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
