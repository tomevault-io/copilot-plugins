## talkflow

> This file provides guidance to Code Agents (Codex, Cursor, etc.) when working with code in this repository.

# AGENTS.md

This file provides guidance to Code Agents (Codex, Cursor, etc.) when working with code in this repository.

## Development Commands

- `pnpm dev` - Start Next.js dev server
- `pnpm build` - Production build
- `pnpm lint` - Biome linter (auto-fixes with `--write`)
- `pnpm format` - Biome formatter
- `pnpm db:generate` - Generate Drizzle migration files from schema changes
- `pnpm db:migrate` - Apply pending migrations
- `pnpm db:push` - Push schema directly to DB (dev only)
- `pnpm db:studio` - Open Drizzle Studio GUI
- `pnpm email` - Email template preview server on port 3333
- `pnpm content` - Rebuild fumadocs MDX content collections
- `pnpm knip` - Find unused exports, dependencies, and files
- `pnpm auth:schema:generate` - Regenerate Better Auth schema to `src/db/auth.schema.ts`

No automated test suite exists. Validate changes with `pnpm build`, `pnpm lint`, and manual QA.

## Architecture Overview

### Routing & i18n
- App Router with `[locale]` dynamic segment using `next-intl` (as-needed prefix strategy — default locale omitted from URL)
- Route groups inside `[locale]`: `(marketing)` for public pages, `(protected)` for authenticated pages, `auth` for login/signup, `docs` for documentation
- API routes at `src/app/api/` (outside locale segment)
- Translation files: `messages/en.json`, `messages/zh.json`
- i18n routing config: `src/i18n/routing.ts`; middleware: `src/middleware.ts`

### Authentication (Better Auth)
- Server config: `src/lib/auth.ts`; client: `src/lib/auth-client.ts`
- PostgreSQL adapter via Drizzle, session cached 60s, 7-day expiry, fresh age disabled
- Plugins: admin, apiKey, emailHarmony (verification/password reset)
- OAuth: GitHub + Google with account linking
- Auth hooks auto-subscribe new users to newsletter and distribute registration credits
- Auth tables in `src/db/schema.ts`: `user`, `session`, `account`, `verification`, `apikey`

### Database (Drizzle ORM + PostgreSQL)
- Connection: `src/db/index.ts` using `postgres` driver (not `pg`), singleton pattern
- Schema: `src/db/schema.ts` — auth tables + `payment`, `userCredit`, `creditTransaction`
- Payment records track `type` (subscription/one-time), `scene` (lifetime/credit/subscription), `status`; unique constraint on `invoiceId`
- Config: `drizzle.config.ts` reads `DATABASE_URL` from env

### Server Actions (next-safe-action)
- Three-tier action clients in `src/lib/safe-action.ts`:
  - `actionClient` — base, no auth required
  - `userActionClient` — requires authenticated session, ctx includes user/session
  - `adminActionClient` — requires admin role
- All actions use Zod schemas for input validation
- Actions organized by feature in `src/actions/`

### Payment System (Stripe)
- Provider pattern in `src/payment/` with Stripe as implementation
- Plans defined in `src/config/website.tsx`: Free (50 credits), Pro ($9.90/mo or $99/yr, 1000 credits), Lifetime ($199 one-time, 1000 credits)
- Credit packages: Basic (100), Standard (200), Premium (500)
- Webhook handler validates signature and distributes credits on payment completion
- Checkout flow: server action → create Stripe customer if needed → record payment → redirect to Stripe → webhook updates record

### Credits System
- Core logic in `src/credits/`
- 7 transaction types: `MONTHLY_REFRESH`, `REGISTER_GIFT`, `PURCHASE_PACKAGE`, `SUBSCRIPTION_RENEWAL`, `LIFETIME_MONTHLY`, `USAGE`, `EXPIRE`
- Credits tracked in `userCredit` table with history in `creditTransaction`
- Monthly distribution via `pnpm distribute-credits` script

### Provider Pattern (used throughout)
All external integrations follow a pluggable provider pattern with factory functions:
- Payment: `src/payment/` (Stripe)
- Mail: `src/mail/` (Resend, React Email templates — all localized)
- Notifications: `src/notification/` (Discord, Feishu)
- Storage: `src/storage/` (S3 via `s3mini`)
- AI: `src/ai/` (multiple image generation providers)

### State & Data Flow
- Server components fetch data directly; mutations via server actions
- Zustand stores in `src/stores/` for client-side state
- React Query for async data on client
- Forms use React Hook Form + Zod validation

### Content
- Fumadocs for documentation (`content/docs/`), MDX blog (`content/blog/`)
- Source config: `source.config.ts`; rebuild with `pnpm content`

### Configuration
- Centralized app config with feature flags: `src/config/website.tsx`
- Demo mode: `NEXT_PUBLIC_DEMO_WEBSITE` env var (enables Crisp chat, Turnstile CAPTCHA, looser admin checks)
- Environment template: `env.example`

## Code Style

- Biome enforces: 2-space indentation, 80-char line width, single quotes, ES5 trailing commas, semicolons required
- Filenames: kebab-case (`dashboard-sidebar.tsx`); hooks prefixed `use-` (`use-session.ts`)
- Named exports preferred; default exports only for pages/layouts
- Server-only code marked with `"use server"` directive
- Tailwind CSS v4 with tokens in `src/styles/`
- UI primitives from Radix UI; icons from `lucide-react`
- Conventional Commits: `feat:`, `fix:`, `chore:`

---
> Source: [zjutimes2015/talkflow](https://github.com/zjutimes2015/talkflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
