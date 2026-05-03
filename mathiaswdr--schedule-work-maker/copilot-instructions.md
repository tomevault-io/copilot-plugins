## schedule-work-maker

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Kronoma (package name: "kronoma") — a work time-tracking SaaS built with Next.js 14 App Router, TypeScript, Prisma/PostgreSQL, and Stripe payments.

## Commands

```bash
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # prisma generate && next build
npm run lint         # next lint
npx prisma generate  # Regenerate Prisma client after schema changes
npx prisma migrate dev --name <name>  # Create and apply a migration
npx prisma db push   # Push schema changes without migration (dev only)
```

Note: `db:generate`, `db:push`, and `push` scripts in package.json reference Drizzle ORM (legacy/unused). Use Prisma commands directly.

## Architecture

### Route Groups (App Router)

- `app/(site)/` — Public pages (landing, login, about, pricing). Uses `layout.tsx` with Nav + Footer.
- `app/(dashboard)/` — Protected dashboard routes (time tracking, stats, settings). Uses `layout.tsx` with Sidebar.
- `app/api/` — API routes: `work-sessions/`, `checkout/`, `webhooks/stripe/`, `auth/[...nextauth]/`, `uploadthing/`.

### Server Layer (`server/`)

- `auth.ts` — NextAuth v5 (beta) config. Google OAuth provider, JWT strategy, PrismaAdapter. Creates Stripe customer on user signup. Access session via `auth()`.
- `prisma.ts` — Prisma client singleton.
- `stripe.ts` — Stripe client instance.
- `work-sessions.ts` — Core business logic: `startSession()`, `pauseSession()`, `resumeSession()`, `endSession()`, `getWorkSummary()`, time calculation helpers (`getSessionMs`, `getBreakMs`).
- `actions/` — Server actions using `next-safe-action` with Zod schemas for type-safe mutations.

### Components

- `components/ui/` — shadcn/ui components (Radix UI + Tailwind + CVA).
- `components/dashboard/` — Time tracking UI, stats, sidebar.
- `components/auth/` — Login form, auth card, social buttons.
- `components/payment/` — Stripe checkout, payment dashboard.
- `components/navigation/` — Nav bar, user button.
- `components/providers/` — Theme provider (next-themes).

### Data Flow

- **Validation:** Zod schemas in `types/` (e.g., `payment-intent-schema.ts`, `settings-schema.ts`).
- **Forms:** React Hook Form + `@hookform/resolvers` + Zod.
- **Server actions:** `next-safe-action` wraps Zod validation around server mutations.
- **Client state:** Zustand store in `lib/client-store.ts` (cart).

### Database (Prisma + PostgreSQL on Supabase)

Key models: `User` (with `UserPlan` enum: FREE/PRO), `WorkSession` (status: RUNNING/PAUSED/ENDED with breaks via `WorkBreak`). Schema at `prisma/schema.prisma`.

Uses connection pooling (`DATABASE_URL`) and direct connection (`DIRECT_URL`) for migrations.

### Internationalization

`next-intl` with cookie-based locale detection. Translation files in `messages/en.json` and `messages/fr.json`. Config in `i18n/request.ts`.

### Styling

Tailwind CSS with custom brand colors (orange, teal, yellow) and CSS variable-based theming. Dark mode via class strategy. Custom breakpoints: `ms` (max 1279px), `mb` (max 1023px). Animations via `framer-motion` and `tailwindcss-animate`.

## Conventions

- Path alias: `@/*` maps to project root.
- Client components use `"use client"` directive; server actions use `"use server"`.
- File naming: kebab-case (e.g., `time-tracking-client.tsx`).
- Auth access: `import { auth } from "@/server/auth"` then `const session = await auth()`.
- Dashboard has a demo user fallback for unauthenticated access.
- Some code comments are in French.

---
> Source: [mathiaswdr/schedule-work-maker](https://github.com/mathiaswdr/schedule-work-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
