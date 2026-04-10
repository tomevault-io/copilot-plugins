## compusum

> - Stack: Next.js App Router, TypeScript, Bun, Prisma, PostgreSQL, Tailwind.

# AGENTS.md

## Project Context

- Stack: Next.js App Router, TypeScript, Bun, Prisma, PostgreSQL, Tailwind.
- Main business areas: catalog, cart, checkout, orders, admin, auth, shipping, customer upsert.
- Important directories:
  - `src/app`: routes, pages, API handlers.
  - `src/components`: UI and admin/store components.
  - `src/lib`: reusable business logic and integrations.
  - `prisma`: schema, migrations, bootstrap, seed, operational validation.

## Review Priorities

Review for correctness, regressions, data safety, and operational readiness first.
Do not spend most of the review on formatting or generic lint suggestions unless they hide a real bug.

Focus on:

1. Checkout, auth, orders, carts, catalog mode, shipping, and admin flows.
2. Prisma alignment between schema, migrations, seed, bootstrap, and runtime startup.
3. Regressions that would break deploy, startup, seeding, or production requests.
4. Missing validation steps when a change touches database or app flow.

## Repository Standards

### Architecture

- Keep API route handlers thin when possible; shared business rules should live in `src/lib`.
- Reuse existing helpers before introducing duplicate logic.
- Preserve the current separation between storefront, admin, and API code.
- Avoid large cross-cutting refactors unless the change clearly requires them.

### TypeScript And React

- Prefer explicit, readable code over clever abstractions.
- Be careful with nullable Prisma fields, auth state, request payloads, and optional customer data.
- In React and route handlers, flag stale assumptions around async data, cookies, params, and server/client boundaries.
- This repo has permissive ESLint settings. Treat lint cleanliness as secondary to behavior and production safety.

### Database And Prisma

If a change touches `prisma/schema.prisma`, `prisma/migrations/**`, `prisma/seed.ts`, `prisma/bootstrap.ts`, startup files, checkout, auth, orders, or any flow that depends on new columns, review it as a high-risk operational change.

Required standards:

1. Schema changes must include a matching migration when runtime code or seed already depends on the new field/table/index.
2. New migration SQL must be idempotent where applicable:
   - use `ADD COLUMN IF NOT EXISTS`
   - use `CREATE INDEX IF NOT EXISTS`
   - use `CREATE UNIQUE INDEX IF NOT EXISTS`
3. If a migration adds unique constraints or unique indexes on existing data, it should include deduplication or normalization before the constraint is created.
4. Seed code must only rely on tables and columns that exist after `prisma migrate deploy`.
5. Do not accept code that updates schema or seed but ignores startup/bootstrap implications.
6. Review `package.json`, `docker-entrypoint.sh`, and `prisma/bootstrap.ts` together when deploy/startup behavior changes.
7. Never suggest blind restarts as a fix for migration failures.
8. Never treat `prisma db push` as a safe replacement for proper migrations in shared or production environments.

Critical operational expectations:

- `prisma/bootstrap.ts` must continue handling:
  - `P3005` by baselining `0_init` when appropriate.
  - `P3009` by resolving the failed migration as rolled back and retrying deploy.
- Reviews should explicitly call out risk of `P2022`, `P3005`, `P3009`, and unique constraint failures when relevant.

### Storefront And Business Rules

- Treat catalog mode as sensitive: prices must not leak into storefront/cart/checkout surfaces when catalog mode is active.
- Order creation must preserve customer linkage, agent assignment, and route assignment behavior when applicable.
- Auth changes must be reviewed for both session/cookie behavior and customer/admin separation.
- Shipping changes must be checked for cutoff logic, city/routing behavior, and message consistency.

## Validation Expectations

When reviewing a change, call out missing validation if the author did not verify the relevant commands.

For Prisma, seed, bootstrap, or startup changes, expect:

1. `bunx prisma generate`
2. `bunx prisma migrate status`
3. `bunx prisma migrate deploy`
4. `bun run seed`

For route, API, component, checkout, auth, or UI flow changes, also expect:

1. `bun run build`

If a validation step was not run, say so explicitly and explain the operational risk.

## How To Report Findings

- Prioritize findings by severity: production breakage, data loss, deploy failure, auth bypass, checkout/order regression, then lower-risk issues.
- Be specific about the root cause, not just the symptom.
- Prefer actionable findings tied to files and behavior.
- Mention when a change is safe but under-validated.
- If no real issues are found, say that clearly and mention any remaining testing or operational gaps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JhonyAlex)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JhonyAlex)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
