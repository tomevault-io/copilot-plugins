## ai-development-agency-mvp-builds

> Build production-ready MVPs for founders and operators using a fixed, reproducible stack. Deliver working software with scope documentation, UAT sign-off, and complete handoff. Not freelance — a product methodology.

# CLAUDE.md — AI Development Agency (MVP Builds)

## Mission

Build production-ready MVPs for founders and operators using a fixed, reproducible stack. Deliver working software with scope documentation, UAT sign-off, and complete handoff. Not freelance — a product methodology.

## Tier Classification

**Tier A — Launch First.** Fixed-price project model eliminates time-tracking overhead. Claude Code dramatically compresses build time. First revenue in 30 days is achievable without employees.

## Tech Stack (Fixed — Do Not Deviate Without Approval)

- **Frontend:** Next.js 14+ (App Router, Server Components, TypeScript)
- **Backend:** Next.js API routes + Supabase (PostgreSQL, Auth, Storage, Realtime)
- **Deployment:** Vercel (frontend + serverless functions) + Supabase Cloud
- **Styling:** Tailwind CSS
- **State:** Zustand (client) or React Context for light state
- **Forms:** React Hook Form + Zod validation
- **Testing:** Vitest + Playwright (E2E for critical user journeys)

**Why fixed stack:** Estimation accuracy requires familiarity. We can estimate a Next.js + Supabase build with < 15% variance. We cannot estimate a build on an unfamiliar stack with any confidence.

## Service Scope

- Discovery and specification (requirements document + wireframes)
- Full-stack web application builds (Next.js + Supabase)
- API integrations (Stripe, Resend, Twilio, third-party APIs)
- Database design and schema
- Authentication and authorization
- Admin dashboards and internal tools
- UAT process management
- Deployment and go-live
- Handoff documentation
- Maintenance retainers (bug fixes, minor features)

## What Claude Can Do Without Asking

- Write scope documents and requirements specifications
- Draft architecture decisions and ERD descriptions
- Write React components and Supabase queries
- Write API route handlers
- Write Zod schemas and validation logic
- Write Tailwind UI components
- Write unit tests and E2E test skeletons
- Write handoff documentation
- Update MEMORY.md, DECISIONS.md, PROGRESS.md, SESSION_HANDOFF.md

## What Requires Human Approval

- Any database migration on a production Supabase instance
- Changing scope after the scope document is signed (change request process)
- Deploying to production (client must confirm UAT sign-off in writing)
- Adding a new package dependency that wasn't in the original scope
- Storing any sensitive user data (PII, payment data) — architecture must be reviewed
- API keys and credentials — never hardcoded, always environment variables

## Critical Quality Rules

- **TypeScript strict mode.** No `any` types.
- **All inputs validated with Zod** at API route boundaries.
- **Environment variables for all secrets.** Never hardcoded. Never committed.
- **UAT before every production deployment.** No exceptions.
- **Change control.** Scope changes require a written change request and additional billing before work begins.
- **No Friday deployments.** Production goes live Tuesday–Thursday.

## Writing Standards (for Code)

- Components are small and focused on one responsibility
- API routes are thin — business logic lives in service functions
- Database queries use parameterized inputs only (no string concatenation)
- Error boundaries at API routes — never expose internal errors to the client
- Comments explain WHY, not WHAT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/findbene) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
