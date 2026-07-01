## prospectkit

> This file provides guidance to Codex when working in this repository.

# AGENTS.md

This file provides guidance to Codex when working in this repository.

## Project Overview

ProspectKit is an open-source outbound prospecting workspace for small teams. It helps users build a company list, research accounts, discover contacts, score leads, draft outreach, and view basic pipeline analytics.

Current status: working prototype. Favor small, direct changes that keep the product loop usable.

**Stack**: Next.js 16 App Router, React 19, Tailwind CSS, shadcn/ui primitives, Drizzle ORM, PostgreSQL, Vercel AI SDK, Exa search.

**Package manager**: pnpm (`pnpm-lock.yaml`). Use `pnpm install` and `pnpm <script>` — not npm.

## Development Commands

```bash
pnpm dev          # Runs DB auto-migration, then starts Next dev server
pnpm build        # Build for production
pnpm start        # Start production server
pnpm lint         # Run ESLint over app, components, and lib
pnpm format       # Run Prettier
pnpm format:check # Check Prettier formatting
```

If `pnpm dev` is blocked by database connectivity while working only on UI, run Next directly:

```bash
./node_modules/.bin/next dev --webpack
```

Webpack mode is useful if Turbopack panics while editing or verifying local UI.

## Database Commands

```bash
pnpm db:auto-migrate # Run migrations automatically
pnpm db:generate     # Generate migration files from schema changes
pnpm db:migrate      # Run migrations manually
pnpm db:push         # Push schema changes directly to DB
pnpm db:studio       # Open Drizzle Studio
pnpm db:check        # Check migration/schema consistency
pnpm db:pull         # Introspect existing database schema
pnpm db:drop         # Drop migration files
pnpm db:seed         # Seed demo companies
pnpm db:seed:force   # Re-seed demo companies even if data exists
```

`DATABASE_MIGRATIONS_URL` is recommended when `DATABASE_URL` points at a pooler.

## Environment

Required:

```bash
DATABASE_URL=
```

Recommended or feature-dependent:

```bash
DATABASE_MIGRATIONS_URL=
EXA_API_KEY=
AI_MODEL_PROVIDER=openai
AI_MODEL_ID=gpt-5.4-mini
```

`AI_MODEL_PROVIDER` supports `openai` in `lib/ai/model.ts`. The default provider is `openai`.

## Application Structure

- `app/(marketing)/page.tsx`: public landing page.
- `app/(workspace)/app/dashboard/page.tsx`: Discover Companies.
- `app/(workspace)/app/contacts/page.tsx`: Manage Leads.
- `app/(workspace)/app/analytics/page.tsx`: Pipeline Analytics.
- `app/(workspace)/app/brief/[companyId]/page.tsx`: Company Prospecting Brief.
- `app/(workspace)/app/settings/scoring/page.tsx`: lead scoring configuration.
- `app/api/reports/generate/route.ts`: AI company report generation.
- `app/api/contacts/find/route.ts`: AI-assisted contact discovery.
- `app/api/leads/score/route.ts`: AI lead scoring.
- `app/api/messages/generate/route.ts`: outreach message generation.
- `app/api/briefs/generate/route.ts`: orchestration route for report, contacts, and scoring.

## Core Product Flow

1. Add or seed companies in `companies`.
2. Generate a company report using Exa search plus the configured chat model.
3. Find likely contacts for the company and save them to `contacts`.
4. Score contacts using the active scoring config.
5. Generate outreach copy from company, contact, report, and score context.
6. Use analytics to view lead status, score distribution, and created-over-time trends.

## Database Schema

Schema files live in `lib/db/schema/`:

- `companies.ts`: account list, priority, website, description, and report status.
- `contacts.ts`: company contacts, status, priority, LinkedIn URL, and notes.
- `company-reports.ts`: structured report JSON and report status.
- `lead-scores.ts`: one score per contact across five dimensions plus reasoning JSON.
- `scoring-config.ts`: single global scoring profile and dimension weights.

Repository helpers live in `lib/server/repositories/`. Server actions used by pages live in `lib/actions/`.

When modifying schema:

1. Update `lib/db/schema/*`.
2. Run `pnpm db:generate`.
3. Run `pnpm db:migrate` or `pnpm dev`.
4. Seed or inspect data with `pnpm db:seed` and `pnpm db:studio`.

## AI and Search

- Model selection is centralized in `lib/ai/model.ts`.
- Exa integration is in `lib/server/services/search/exa.ts`.
- Company reports are generated in `lib/server/services/reports/generate-company-report.ts`.
- Contact discovery is in `lib/server/services/contacts/find-contacts-for-company.ts`.
- Lead scoring is in `lib/server/services/leads/score-lead-for-contact.ts`.
- Outreach generation is in `lib/server/services/messages/generate-outreach-message.ts`.

Service functions return `Result` objects from `lib/server/shared/result.ts`; API routes convert these through `lib/server/http/error-response.ts`.

## Frontend Notes

- Shared UI primitives are in `components/shared/ui/`.
- Workspace components are grouped by domain under `components/workspace/`.
- Marketing content is centralized in `lib/marketing/landing-content.ts`.
- Marketing logo assets live in `public/marketing/logos/`.
- Prefer existing shadcn/ui patterns and lucide icons before adding new UI primitives.
- Keep operational workspace screens dense and scannable; avoid marketing-style layouts inside the app shell.

## Verification

For most changes:

```bash
pnpm lint
```

For schema or TypeScript-heavy changes, also run:

```bash
./node_modules/.bin/tsc --noEmit
```

For UI work, run a local dev server and verify the affected route in the in-app browser.

---
> Source: [zebangeth/ProspectKit](https://github.com/zebangeth/ProspectKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
