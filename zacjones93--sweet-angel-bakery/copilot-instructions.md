## sweet-angel-bakery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack e-commerce bakery platform (Sweet Angel Bakery) built on Cloudflare Workers with Next.js 15. Includes customer storefront, admin management, delivery tracking, loyalty program, and Square/Stripe payment processing.

**Tech Stack**: Next.js 15 (App Router) • React 19 • TypeScript • Cloudflare Workers (OpenNext) • Drizzle ORM • D1 Database • Tailwind CSS 4 • Shadcn UI

## Essential Commands

### Development
```bash
pnpm dev                    # Start dev server (localhost:3000)
pnpm db:migrate:dev         # Apply migrations to local SQLite
pnpm db:studio              # Open Drizzle Studio (visual DB explorer)
pnpm email:dev              # Preview email templates (localhost:3001)
```

### Database
```bash
pnpm db:generate [NAME]     # Generate migration from schema changes
pnpm db:migrate:prod        # Apply migrations to production D1
```

### Build & Deploy
```bash
pnpm build                  # Next.js build
pnpm deploy                 # Build with OpenNext + deploy to Cloudflare
pnpm cf-typegen             # Generate Cloudflare binding types (after wrangler.jsonc changes)
```

### Utilities
```bash
pnpm scrape:products        # Scrape products from external source
pnpm sync:square            # Sync with Square API
pnpm import:square          # Import products from Square
```

## Architecture

### Route Structure
- `(auth)/*` - Email/password, WebAuthn, Google OAuth flows
- `(storefront)/*` - Customer-facing pages (products, cart, checkout, loyalty signup)
- `(admin)/admin/*` - Admin dashboard (orders, products, users, analytics)
- `(settings)/settings/*` - User profile and session management
- `(legal)/*` - Terms, privacy policy
- `api/*` - API routes, webhooks (Square/Stripe), image uploads

### Key Architectural Patterns

**Merchant Provider Abstraction**: Payment processor abstraction layer in `src/lib/merchant-provider/`. Factory pattern selects Square or Stripe dynamically. To add new provider, implement interface in `providers/` and update factory.

**Type-Safe Server Actions (ZSA)**: All server operations use `createServerAction()` with Zod validation. Actions co-located with routes in `*.actions.ts` files. Example:
```typescript
export const myAction = createServerAction()
  .input(mySchema)
  .handler(async ({ input }) => { /* ... */ });
```

**Session Storage in KV**: Sessions stored in Cloudflare KV (not D1) for edge performance. See `src/utils/kv-session.ts`. Max 5 sessions/user, 30-day expiration.

**Delivery System**: Timezone-aware delivery date calculations (`src/utils/timezone.ts`), route optimization with Google Maps, ETA tracking.

**Product Customizations**: Flexible addon system with price adjustments. Schema in `src/schemas/customizations.schema.ts`, stored as JSON in order items.

### Database (Drizzle ORM + D1)

**Schema**: `src/db/schema.ts` - 21+ tables including users, products, orders, delivery, loyalty, inventory
**Critical**: Never use Drizzle transactions (D1 doesn't support). Never pass `id` to insert/update (auto-generated CUID2).
**Migration workflow**:
1. Edit `src/db/schema.ts`
2. Run `pnpm db:generate migration-name`
3. Apply with `pnpm db:migrate:dev` (local) or `pnpm db:migrate:prod` (production)

**DB Access Pattern**:
```typescript
const db = getDB(); // Cached via React cache()
await db.query.userTable.findFirst({ where: eq(...) });
```

### Authentication

**Methods**: Email/password (bcrypt) • WebAuthn/Passkeys (@simplewebauthn) • Google OAuth (Arctic v3) • Magic links (storefront)

**Session Access**:
- Server Components: `getSessionFromCookie()` from `src/utils/auth.ts`
- Client Components: `useSessionStore()` from `src/state/session.ts`

**Key Files**:
- `src/utils/auth.ts` - Session creation/validation
- `src/utils/kv-session.ts` - KV storage logic
- `src/lib/sso/google-sso.ts` - Google OAuth setup

### Cloudflare Bindings (wrangler.jsonc)

- `NEXT_TAG_CACHE_D1` - D1 database for tag-based cache
- `NEXT_INC_CACHE_KV` - KV for incremental cache (ISR)
- `PRODUCT_IMAGES` - R2 bucket for images
- `NEXT_CACHE_DO_QUEUE` - Durable Object for cache queue

**After editing wrangler.jsonc**: Always run `pnpm cf-typegen`

## Code Conventions

### General
- Functional components, TypeScript, no classes
- Server Components by default; add `"use client"` only when needed
- Add `import "server-only"` at top of server-only files (except page.tsx)
- Functions with >1 param: use named object `function foo({ a, b }) {}`
- Never delete comments unless irrelevant

### Imports
- Check `package.json` before adding packages (avoid duplicates)
- Always use `pnpm` for package management
- Server actions: `import { useServerAction } from "zsa-react"`

### State Management
- Server state: React Server Components
- Client state: Zustand (`src/state/`)
- URL state: NUQS
- Forms: React Hook Form + Zod

### Types
- Add global types to `custom-env.d.ts` (not `cloudflare-env.d.ts` - gets overwritten)
- Use `@/*` path alias for imports

### Database
- Never pass `id` to `.insert().values()` or `.update()` (auto-generated)
- No transactions (D1 limitation)

### Styling
- Tailwind CSS 4 with Shadcn UI components (`src/components/ui/`)
- Dark/light mode via next-themes

## Environment Setup

1. `pnpm install`
2. Copy `.dev.vars.example` → `.dev.vars` (Cloudflare credentials, API keys)
3. Copy `.env.example` → `.env` (public keys: Turnstile, Square, Stripe, Google OAuth, email service)
4. `pnpm db:migrate:dev` - Create local DB
5. `pnpm dev`

## Deployment (GitHub Actions)

CI/CD in `.github/workflows/deploy.yml` triggers on push to `main`:
1. Checkout, setup pnpm/Node.js 23
2. Install deps, migrate local D1
3. `pnpm deploy` (OpenNext build + Cloudflare deploy)
4. Apply remote D1 migrations
5. Purge CDN cache

## Important Notes

- **No test suite**: Project uses Playwright for manual/browser testing
- **Webhook verification**: HMAC-SHA256 for Square webhooks in `api/webhooks/square/route.ts`
- **Rate limiting**: IP-based via `withRateLimit()` decorator (Cloudflare KV)
- **Email templates**: React Email components in `src/react-email/` (Resend or Brevo)
- **Security**: Turnstile CAPTCHA, anti-disposable email validation, input sanitization via Zod
- **Scripts**: Product scraping, Square sync utilities in `scripts/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zacjones93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
