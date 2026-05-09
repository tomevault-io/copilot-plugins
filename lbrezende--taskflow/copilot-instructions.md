## taskflow

> This is a Next.js 14+ SaaS MVP boilerplate with authentication, payments, and a task management product.

# TaskFlow — Claude Code Context

This is a Next.js 14+ SaaS MVP boilerplate with authentication, payments, and a task management product.

## Project Conventions

- **Framework:** Next.js App Router (NO src/ directory)
- **Language:** TypeScript strict mode
- **Styling:** Tailwind CSS 4 + shadcn/ui components
- **ORM:** Prisma 6 with PostgreSQL (Neon)
- **Auth:** Auth.js v5 (NextAuth) — Google OAuth + Email Magic Link
- **Payments:** Stripe subscriptions with trial support
- **Data fetching:** TanStack Query (client), Server Components (server)
- **Validation:** Zod schemas in `lib/validations.ts`
- **Email:** Resend

## Architecture Rules

1. **Route Groups:** `(public)` for unauthenticated pages, `(auth)` for protected pages
2. **API Routes:** Always validate auth with `auth()` from `lib/auth.ts`
3. **Subscription Logic:** All plan checks go through `lib/subscription.ts`
4. **Paywall:** Use `<PaywallGate>` component for feature gating
5. **3-Layer Auth Protection:** middleware → server component check → client component check

## Key Files

- `lib/auth.ts` — NextAuth config, JWT callbacks, user creation with trial
- `lib/subscription.ts` — Plan limits, trial/subscription checks
- `lib/stripe.ts` — Checkout sessions, customer portal
- `lib/validations.ts` — Zod schemas for all inputs
- `prisma/schema.prisma` — Database schema
- `middleware.ts` — Route protection

## Plan System

- **FREE:** Limited features (defined in PLAN_LIMITS)
- **TRIAL:** 14 days, unlimited access, starts on first signup
- **PRO:** Paid via Stripe, unlimited access
- **Upgrade during trial:** Users can upgrade immediately during trial. Stripe checkout does NOT set `trial_period_days`, so payment is immediate. Webhook sets `plan: "PRO"`.

## First-Time Setup Guide

When a new user clones this repo and runs Claude Code for the first time, guide them through setup if `.env` is missing or empty. Follow this order:

1. `cp .env.example .env`
2. **Neon** — Create project at neon.tech, copy connection string → `DATABASE_URL`
3. **Google OAuth** — Create OAuth 2.0 credentials at console.cloud.google.com, redirect URI: `http://localhost:3000/api/auth/callback/google` → `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`
4. **Resend** — Create API key at resend.com → `AUTH_RESEND_KEY`, `RESEND_API_KEY` (same key)
5. **Stripe** — Copy secret key → `STRIPE_SECRET_KEY`. Create product with recurring price → `STRIPE_PRICE_ID_PRO`. Run `stripe listen --forward-to localhost:3000/api/stripe/webhook` → `STRIPE_WEBHOOK_SECRET`
6. **AUTH_SECRET** — Run `openssl rand -base64 32`
7. Run `npx prisma generate && npx prisma db push`
8. Run `npm run dev`

After setup, suggest connecting Figma MCP for design-to-code workflows.

## Figma MCP Integration

To enable design-to-code with Figma:

1. Add to Claude Code MCP settings:
   ```json
   {
     "mcpServers": {
       "figma": {
         "command": "npx",
         "args": ["-y", "@anthropic-ai/figma-mcp-server@latest"]
       }
     }
   }
   ```
2. Authenticate with Figma account when prompted
3. Use `get_design_context` with Figma node URLs to extract pixel-perfect code
4. Use `get_screenshot` to visually verify designs

## Stripe SDK v20 Notes

- `current_period_end` was removed from Subscription object. Use `invoice.period_end` via `latest_invoice`.
- `invoice.subscription` moved to `invoice.parent.subscription_details.subscription`.
- API version: `2026-02-25.clover`.
- Stripe client uses lazy Proxy pattern in `lib/stripe.ts` to avoid build-time crashes when env vars are missing.

## Middleware Constraints

- Edge Function limit: < 1MB on Vercel free tier
- NEVER import `auth` from Auth.js in middleware (pulls Prisma+Resend+providers ~1.01MB)
- Middleware checks `authjs.session-token` cookie directly instead

---
> Source: [lbrezende/taskflow](https://github.com/lbrezende/taskflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
