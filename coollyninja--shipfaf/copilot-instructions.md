## shipfaf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShipFree is an open-source SaaS boilerplate alternative to ShipFast, built with Next.js 16, TypeScript, and modern web technologies. It provides authentication, payment processing (Stripe & LemonSqueezy), email notifications, and database integration out of the box.

## Development Commands

**Package Manager**: This project uses `pnpm` (version 9.15.3+)

- `pnpm dev` - Start development server with Turbopack
- `pnpm build` - Build production bundle
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint
- `pnpm format` - Format code with Prettier

**Database (Drizzle ORM)**:
- Database schema is defined in `src/db/schema.ts`
- Drizzle config is in `drizzle.config.ts`
- Uses PostgreSQL as the primary database

**Docker**:
- Dev (watch mode): `docker-compose -f docker/dev/docker-compose.yml -f docker/dev/docker-compose.postgres.yml up --build`
- Prod: `docker-compose -f docker/prod/docker-compose.yml -f docker/prod/docker-compose.postgres.yml up --build -d`
- Portainer available at `http://localhost:9000`

## Architecture

### App Structure (Next.js App Router)

- `src/app/(site)/` - Public marketing pages (Hero, Features, Pricing, Testimonials, etc.)
- `src/app/auth/` - Authentication pages (login, register, email confirmation)
- `src/app/dashboard/` - Protected dashboard pages
- `src/app/api/` - API routes for webhooks and server actions
  - `api/stripe/` - Stripe checkout and webhooks
  - `api/lemonsqueezy/` - LemonSqueezy webhooks
  - `api/mailgun/` - Email handling

### Authentication Flow

**Provider**: Supabase Auth with SSR
- Client: `src/lib/supabase/client.ts` - For client components
- Server: `src/lib/supabase/server.ts` - For server components/actions (uses cookie store)
- Middleware: `src/lib/supabase/middleware.ts` - Session management and protected route handling

**Important**: The middleware redirects unauthenticated users to `/login` for protected routes. Always use `createClient()` with proper cookie handling in server components.

### Payment Integration

**Dual Payment Providers** (Stripe & LemonSqueezy):

1. **Stripe**:
   - Client SDK in `src/utils/stripe.ts` (singleton pattern with `getStripe()`)
   - Webhook handler: `src/app/api/stripe/webhook/route.ts`
   - Handles: subscription created, updated, deleted events

2. **LemonSqueezy**:
   - Client in `src/utils/lemon.ts`
   - Webhook handler: `src/app/api/lemonsqueezy/webhook/route.ts`
   - Uses axios-based API client

**Checkout Components**:
- `src/components/CheckoutButton.tsx` - Stripe checkout
- `src/components/lemon-button.tsx` - LemonSqueezy checkout

### Email System

**Provider**: Mailgun
- Configuration in `src/config.ts`
- Send function in `src/lib/mailgun.ts` with `sendEmail()` helper
- Supports custom subdomain and reply-to handling
- Uses both `fromNoReply` and `fromAdmin` addresses

### Configuration

**Central Config**: `src/config.ts` contains app-wide settings:
- App name and domain
- Mailgun email addresses and subdomain
- Support email forwarding

**Environment Variables** (see `.env.example`):
- Supabase: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- Stripe: `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`
- LemonSqueezy: `LEMON_SQUEEZY_API_KEY`, `LEMON_SQUEEZY_STORE_ID`
- Mailgun: `MAILGUN_API_KEY`, `MAILGUN_DOMAIN`, `MAILGUN_FROM_EMAIL`

### UI Components

**Shadcn/ui**: Components in `src/components/ui/` (Avatar, Button, Card, Input, Label, Tabs)
- Uses Radix UI primitives
- Styled with Tailwind CSS and `class-variance-authority`
- Utility function `cn()` in `src/lib/utils.ts` for conditional classes

**Typography**: Bricolage Grotesque font loaded in root layout

### Important Notes

- **TypeScript**: Build errors are ignored via `typescript: { ignoreBuildErrors: true }` in `next.config.ts`
- **Path Alias**: `@/*` maps to `./src/*`
- **Security Headers**: HSTS, X-Frame-Options, X-Content-Type-Options, and Referrer-Policy configured in Next.js config
- **Page Extensions**: `.ts`, `.tsx`, `.mdx` only

### Database Schema

Currently minimal schema with `usersTable` (id, name, age, email) - likely needs expansion for production use with subscription management.

---
> Source: [coollyninja/shipfaf](https://github.com/coollyninja/shipfaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
