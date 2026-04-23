## koin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
It acts as the primary "Memory" for the agent.

## Project Memory

@llms.txt
@llms-full.txt

## Commands

```bash
pnpm dev        # Start dev server
pnpm build      # Build for production
pnpm lint:fix   # Fix ESLint issues
pnpm format     # Format with Prettier
pnpm typecheck  # TypeScript type checking — run this to verify changes
```

There is no test suite. `pnpm typecheck` is the primary verification step.

## Architecture

Koin is a **Nuxt 4** SPA for expense tracking, tax reporting, and Mercury bank CSV import. All data persists in **SQLite via NuxtHub** — no auth, no external backend.

**Data flow:** Components → composable methods → `$fetch()` → `server/api/` endpoints → Drizzle ORM → NuxtHub SQLite

```
app/
├── pages/index.vue       # Single page, 5 tabs
├── components/           # One component per tab + modals
├── composables/          # All client state (see below)
└── utils/formatters.ts   # Currency, date, CSV helpers

server/
├── api/                  # REST endpoints: expenses, mileage, contractors, recurring, settings, estimated-taxes
└── db/schema.ts          # 6 Drizzle tables; migrations/ alongside
```

## State Management

All client state lives in **module-level `ref()`s inside composables** — no Pinia. Key composables:

- **`useExpenseStore`** — primary store; CRUD for expenses + mileage; `getTaxDefaultsForCategory()`, `getNetDeductible()`, `hasDuplicateExpense()`; computed YTD/category/vendor breakdowns
- **`useMercuryImport`** — parses Mercury CSV; maps categories (22 rules); vendor→category keyword matching (100+ patterns); transfer detection; duplicate flagging before bulk import
- **`useRecurring`** — `processAutoAdd()` auto-creates expenses when `nextDueDate <= today`, runs on app init
- **`useConfirm`** — promise-based confirmation dialog: `confirm(options): Promise<boolean>`

Each composable has `sanitize*()` functions to normalize raw API responses before storing in refs.

## Database

Schema is in `server/db/schema.ts`; migrations in `server/db/migrations/`.

- **Database Dialect**: The database dialect is set in the `nuxt.config.ts` file, within the `hub.db` option or `hub.db.dialect` property.
- **Drizzle Config**: Don't generate the `drizzle.config.ts` file manually, it is generated automatically by NuxtHub.
- **Generate Migrations**: Use `npx nuxt db generate` to automatically generate database migrations from schema changes
- **Never Write Manual Migrations**: Do not manually create SQL migration files in the `server/db/migrations/` directory
- **Workflow**:
  1. Create or modify the database schema in `server/db/schema.ts` or any other schema file in the `server/db/schema/` directory
  2. Run `npx nuxt db generate` to generate the migration
  3. Run `npx nuxt db migrate` to apply the migration to the database, or run `npx nuxt dev` to apply the migration during development
- **Access the database**: Use the `db` instance from `@nuxthub/db` (or `hub:db` for backwards compatibility) to query the database, it is a Drizzle ORM instance.

## Code Style

Prettier: **no semicolons**, single quotes, 100-char line width, no trailing commas, `tailwindcss` plugin sorts classes. ESLint extends `@nuxt/eslint`; `vue/multi-word-component-names` is disabled.

## Commit Style

Follow [Conventional Commits](https://www.conventionalcommits.org):

- `feat:` — new feature
- `fix:` — bug fix
- `docs:` — documentation only
- `chore:` — maintenance, dependency updates, tooling

Examples: `feat: add CSV export`, `fix: correct mileage deduction rounding`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nsina) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
