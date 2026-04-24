## preds-match

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run dev          # Start dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm run db:seed      # Seed database with 3 World Cup matches (prisma/seed.ts)
npx prisma migrate dev --name <name>   # Create and apply migration
npx prisma generate  # Regenerate Prisma client (also runs on postinstall)
```

## Architecture

**Predict the Match** — a World Cup 2026 prediction game (Group I, Norway). Users predict match outcomes (Team1 Win / Draw / Team2 Win), correct predictions earn promo codes sent via email.

### Data Flow Pattern

Server components (pages) fetch data from Prisma, serialize it to plain objects, and pass to client components as props. Client components handle interactivity (countdown timers, prediction buttons, forms) and call API routes for mutations, then `router.refresh()` to re-render server components with fresh data.

### Prisma v7 — Adapter Required

PrismaClient **must** be instantiated with a `PrismaPg` adapter — `new PrismaClient({ adapter })`. This applies everywhere including standalone scripts (see `prisma/seed.ts`). The generated client lives at `src/generated/prisma/client` (not a barrel `index.ts`), so imports use `@/generated/prisma/client`.

### Auth

Cookie-based, no passwords for users. `POST /api/auth/login` with `{email, name}` upserts a user and sets an httpOnly `userId` cookie. Server-side auth check: `getCurrentUser()` from `lib/auth.ts`. Admin panel uses a separate `admin_session` cookie validated against `ADMIN_PASSWORD` env var.

### Cron Job

`/api/cron/check-results` runs every 5 minutes (vercel.json). Requires `Authorization: Bearer ${CRON_SECRET}` header. Fetches results from football-data.org API, scores predictions, generates promo codes (`WC26-XXXXXX`), sends emails via Resend. Falls back to `MANUAL_REVIEW` status after 6 hours without API result.

### Email (Resend)

`lib/email.ts` lazily initializes the Resend client — the app works without `RESEND_API_KEY` (logs warnings instead of crashing). Two email types: promo code delivery and admin alerts.

## Key Constraints

- **One prediction per user per match** — enforced by `@@unique([userId, matchId])` in Prisma and checked in the API route.
- **Prediction deadline** — 1 hour before `matchDate`. Enforced server-side in `POST /api/predictions`.
- **Tailwind v4** — config is in `globals.css` via `@theme inline`, not a separate `tailwind.config.js`.
- **Dark theme only** — `<html>` always has `dark` class. All colors are set on `:root` directly.
- **Fonts** — Bricolage Grotesque (sans, headings) + Instrument Serif (editorial accents), loaded via `next/font/google`.

## Environment Variables

`DATABASE_URL`, `ADMIN_PASSWORD`, `CRON_SECRET` are required. `FOOTBALL_API_KEY`, `RESEND_API_KEY`, `EMAIL_FROM`, `ADMIN_EMAIL` are optional (graceful degradation). See `.env.example`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Funkersss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
