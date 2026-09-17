## julex-refined

> Next.js 14 (App Router) e-commerce platform for **24julex** (anti-tarnish jewelry), hybrid B2C + B2B with role-based access. **`CLAUDE.md` is the full architecture reference — read it before non-trivial work.** This file only captures the rules most likely to be missed.

# AGENTS.md

Next.js 14 (App Router) e-commerce platform for **24julex** (anti-tarnish jewelry), hybrid B2C + B2B with role-based access. **`CLAUDE.md` is the full architecture reference — read it before non-trivial work.** This file only captures the rules most likely to be missed.

## Commands

```bash
npm run dev          # dev server on port 3001
npm run dev:3000     # dev server on port 3000
npm run build        # prisma generate && prisma db push && next build
npm run start        # prod server on port 3001
npm run lint         # next lint
npm run db:push      # prisma db push (schema → DB, dev)
npm run db:generate  # prisma generate
npm run db:migrate   # prisma migrate dev
npm run db:reset     # prisma migrate reset (dev only)
```

Gotchas:
- **`npm run build` mutates the target database schema** (`prisma db push`). Confirm `DATABASE_URL` points at the intended DB before building. `postinstall` also runs `prisma generate`.
- `next.config.js` sets `ignoreBuildErrors: true` — TypeScript errors do **not** fail the build.
- **No test suite / no test runner** is configured. Don't assume `npm test` exists.
- **VPS deploy path is `/root/24julex`, NOT `/var/www/24julex`.** The PM2 process `julex.shop` (id 0) runs with `pm_cwd = /root/24julex` on port 3001. `/var/www/24julex` is a stale, dirty checkout (170+ commits behind, missing platform env vars) — deploying there changes nothing user-facing. Always `cd /root/24julex` before `git pull` / `control:push` / `build` / `pm2 restart`. The multi-DB deploy needs separate pushes: `npm run build` (main schema), `npm run control:push` (control schema), `npm run tenants:migrate` (tenant schema).

## Layer & import rules

- **Prisma client**: `import { db } from '@/lib/db'`. The singleton is exported as `db` — **never** `prisma`, and never import directly from `@prisma/client` (the `Prisma` namespace for types is the only exception).
- **Path alias**: `@/*` → `src/*` (see `tsconfig.json`).
- **API route shape**: handlers return `{ success: boolean, data?: any, error?: string }` and authenticate via `getCurrentUser()` from `@/lib/auth`. Prefer `withErrorHandler()` / `handleApiError()` from `@/lib/errors` for consistent responses.
- **Roles**: Customer (B2C, `priceB2c`) / Reseller (B2B, `priceB2b`, requires `isVerified` on the `Reseller` model) / Admin (portal at `/admin`). The cart store auto-switches pricing on the `isB2B` flag.
- Auth is JWT in an HttpOnly cookie (7-day expiry); Google OAuth and phone/email OTP paths all converge on the same app JWT.

## Sensitive areas — read before editing

- **Schema**: `prisma/schema.prisma` (PostgreSQL; ~15 models; `images`/`features` use native `Json?`).
- **Image uploads**: `src/lib/blob.ts` (Vercel Blob primary, local filesystem fallback for dev). Product images and avatars flow through `/api/products*` and `/api/user/avatar`. Note: `next.config.js` sets `images.unoptimized: true` — the Next.js image optimizer is **off** (disabled for VPS deployment), so don't assume AVIF/WebP/caching runs.
- **Payments**: Razorpay via `/api/payments/*` — `create-order`, `create-link`, `verify`, and `webhook` (signature-verified with `RAZORPAY_WEBHOOK_SECRET`).
- **Deploy/proxy config**: ~15 root docs exist — entry points are `README-DEPLOYMENT.md` and `QUICK_DEPLOY.md`; full guides/checklists include `DEPLOYMENT.md`, `DEPLOYMENT_GUIDE.md`, `DEPLOYMENT-CHECKLIST.md`, `DEPLOYMENT-CHECKLIST-NGINX.md`, `NGINX-DEPLOYMENT.md`, `CADDY-TO-NGINX.md`, `PRODUCTION_SETUP.md`, `PRODUCTION_READY_SUMMARY.md`, `MIGRATION_GUIDE.md`, `SECURITY-CHANGES.md`. Live config: `Caddyfile`, `nginx.conf`, `ecosystem.config.js` (PM2), `deploy.sh`. Read the relevant doc **and** the live config file before touching deployment or reverse-proxy settings.
- **Gathering feature** (stall lead-capture + auto-coupon, merged via `feat/gathering-lead-capture`): admin UI at `src/app/admin/gathering`, API at `src/app/api/gathering` (incl. `validate/`). Coupons are redeemed at **payment confirmation** (not checkout) and `usedCount` is guarded atomically with a per-user redemption check — preserve both guards when editing.

## Environment

- Node.js `>=18.18.2`.
- Only 5 vars are hard-required (validated by `src/lib/config.ts`, fails fast in production): `DATABASE_URL`, `JWT_SECRET` (>=32 chars prod), `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `NEXT_PUBLIC_APP_URL`.
- Everything else (Blob, Nodemailer/SMTP, MSG91 SMS, Web Push VAPID, Firebase) is **feature-gated and degrades silently when unset**. Don't assume an integration is wired just because code references it.
- Config validation rejects default/insecure secret values in production; in dev it logs warnings and continues with fallbacks.

---
> Source: [Nehal-1826/julex-refined](https://github.com/Nehal-1826/julex-refined) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
