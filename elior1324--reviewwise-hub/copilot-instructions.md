## reviewwise-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Identity

**Stack:** React 18 + TypeScript + Vite 5 — this is a **Single Page Application (SPA)**.
**Routing:** React Router v6 (all routes in `src/App.tsx`)
**Styling:** Tailwind CSS 3 + shadcn/ui (Radix UI primitives in `src/components/ui/`)
**Server-state:** TanStack React Query 5
**Global state:** Context API only — `AuthContext` (auth + MFA + rate limiting + subscription) and `ModeContext` (user/business toggle)
**Forms:** React Hook Form + Zod — no exceptions
**Backend:** Supabase (PostgreSQL + RLS + Auth + 39 Edge Functions in Deno TypeScript)
**Payments:** Coupon-based access system for premium tiers. No active payment provider — Grow/HYP have been removed.
**Hosting:** Lovable/Netlify CDN
**UI language:** Hebrew-first, full RTL (`lang="he"`, `dir="rtl"`)
**Supabase project:** `pujsopidbejeuqteormi` (region: ap-southeast-1)

**This is NOT a Next.js project.** There is no App Router, no Server Components, no SSR, no `getServerSideProps`, no `getStaticProps`, no file-based routing. Do not apply Next.js patterns.

## Build & Development Commands

```bash
npm run dev              # Start Vite dev server (http://localhost:8080)
npm run build            # Production build
npx tsc --noEmit         # Type check without emitting
npm run lint             # ESLint
npm run test             # Run all unit tests (Vitest)
npm run test:watch       # Watch mode
npm run test:coverage    # Unit tests with V8 coverage report
npm run test:all         # Run unit tests + E2E tests sequentially
npm run test:e2e         # Playwright E2E tests
npm run test:e2e:headed  # E2E with browser visible
npm run test:e2e:debug   # E2E with Playwright inspector
```

**Running a single test:**
```bash
npx vitest run src/lib/sanitize     # Unit: run tests matching a path/name
npx playwright test auth.spec.ts    # E2E: run a specific spec file
```

**Test file locations:**
- Unit tests: colocated in `src/` as `*.test.ts` / `*.test.tsx` (Vitest + jsdom + Testing Library)
- E2E tests: `tests/e2e/*.spec.ts` (Playwright, Chromium + Mobile Safari)
- Test setup (global mocks for jsdom, framer-motion, sonner, router): `src/test/setup.ts`

**Path alias:** `@/` resolves to `src/` (configured in tsconfig + vite + vitest).

**Supabase CLI:**
```bash
npx supabase gen types typescript --linked > src/integrations/supabase/types.ts  # Regenerate DB types
npx supabase db push                                                              # Push migrations
npx supabase functions deploy <name>                                              # Deploy Edge Function
```

---

## Absolute Rules

### Diagnosis Before Action

- Never claim a fix without tracing the root cause to a specific function, line, data state, or policy. "It should work now" is not a valid conclusion.
- Never describe a change as complete unless you have traced the full behavior path — from user action through component → hook → Supabase call → DB → response → UI update.
- Never silently follow documentation when the code disagrees. `SYSTEM_COHERENCE.md` and `README.md` are reference material, not authority. The implemented code is the source of truth. If there is a conflict, note it explicitly and follow the code.
- Never mark an issue resolved based on "the code looks correct." Verify by tracing a real scenario.

### Scope Discipline

- Never rewrite files unrelated to the current task. If you notice a problem in a different file, note it — do not silently fix it.
- Never rename symbols, reorganize imports, or reformat code unless explicitly asked.
- Never add comments, docstrings, or type annotations to code you did not change.
- Never bundle unrelated cleanup into a task branch.

### Security Rules

- Never weaken any security control without a documented replacement in the same change:
  - Do not remove or bypass rate limiting (client `ClientRateLimiter` + server `check_login_rate_limit` RPC)
  - Do not relax RLS policies without verifying all affected access paths
  - Do not disable or skip Cloudflare Turnstile CAPTCHA
  - Do not loosen CSP headers in `vite.config.ts` or `public/_headers`
  - Do not change token storage from `sessionStorage` to `localStorage`
  - Do not remove session timeout logic (30-minute idle, 25-minute warning)
- Never hardcode secrets, API keys, or credentials anywhere. Frontend uses `import.meta.env.VITE_*`. Edge Functions use `Deno.env.get()`. Server-side secrets (`RESEND_API_KEY`, `TURNSTILE_SECRET_KEY`, etc.) must never appear in the frontend bundle.
- Never use the Supabase service role key in frontend code. It belongs only in Edge Functions for privileged operations.
- Never bypass RLS by switching to the service role client from frontend components.
- When integrating a future payment provider, always verify webhook HMAC signatures before processing. No payment provider is currently active.

### Trust Score Isolation — Critical Architectural Rule

The trust system has three strictly isolated layers. Never mix them.

| Layer | Table | Type | Redeemable | Affects Trust Score |
|---|---|---|---|---|
| Trust Score | `businesses.trust_score` | 0–100 business metric | N/A | Yes — this IS the score |
| Activity Points | `user_points` | Redeemable currency | Yes (600 pts = 20% discount) | No |
| Community Points | `leaderboard_entries` | Seasonal reputation | No — zero cash value | No |

- The Trust Score is not purchasable, not influenced by Activity Points or Community Points, and not affected by referral counts, leaderboard rank, or subscription tier.
- Activity Points and Community Points are independent of each other — a change to one must never alter the other.
- When modifying any of these systems, verify you are touching the correct table and the correct layer before proceeding.
- Functions that update Trust Score must not read from `user_points` or `leaderboard_entries`.
- Functions that grant Activity Points or Community Points must not write to `businesses.trust_score`.

### Checkout & Subscription Architecture

- **No active payment provider.** Grow and HYP (YaadPay) have been removed. Do not introduce a payment provider without explicit discussion.
- **Coupon system** (`apply-coupon` Edge Function) handles premium access codes. Coupons are in `public.coupons` table, redemptions in `public.coupon_redemptions`.
- `duration_months = 0` means **lifetime** access (no expiration). `subscription_expires_at = NULL` on a non-free tier = lifetime.
- `check-subscription` Edge Function determines active tier. It auto-downgrades to "free" when `subscription_expires_at` has passed, but **never** downgrades lifetime subscriptions (NULL expiry).
- Three subscription tiers: `"free"` | `"pro"` | `"enterprise"`. Feature gating is in `src/hooks/useFeatureGating.ts`.
- Platform is currently 100% free — `checkSubscription()` in AuthContext returns `"enterprise"` for all users. When a payment provider is added, this must be replaced with a real server-side check.

### Auth & Sessions

- All auth mutations route through `AuthContext` (`src/contexts/AuthContext.tsx`). Never call Supabase auth methods directly from page or component files.
- Token storage is `sessionStorage` — intentional (clears on tab close). Do not change to `localStorage` without an explicit security review.
- Session timeout is 30 minutes idle with a warning at 25 minutes. Do not reduce or remove this.
- PKCE flow is required for OAuth. Do not downgrade to implicit grant.
- If `mfaRequired: true` is returned from `signIn()`, the UI must gate access until the TOTP challenge is completed. Do not short-circuit MFA.
- When changing login flow, test all error paths: wrong password, locked account (5 failures / 15 min), MFA challenge, expired session, OAuth failure.

### Database & Migrations

- Never modify existing migration files in `supabase/migrations/`. They represent applied history.
- Every schema change requires a new migration file (format: `YYYYMMDDHHMMSS_description.sql`).
- After any migration that adds or removes columns, regenerate types:
  ```
  supabase gen types typescript --linked > src/integrations/supabase/types.ts
  ```
  Never hand-edit `src/integrations/supabase/types.ts`.
- Test RLS policy changes against all relevant roles (`anon`, `authenticated`, service role) before deploying.

### Architecture Consistency

- This is a React SPA with React Router v6. Do not introduce Next.js routing, file-based routing, SSR assumptions, or patterns from Remix.
- SEO is static-only: one HTML file, static meta tags in `index.html`. This is a known SPA limitation. Do not claim per-page SEO without implementing a real solution (prerendering, react-helmet-async, or SSG). The current implementation has no per-page dynamic meta management.
- New routes go in `src/App.tsx` — do not create shadow routing systems.
- State management is React Query (server-state) + Context API (global client-state). Do not introduce Redux, Zustand, Jotai, or other global state libraries without explicit discussion.
- Do not introduce server-side rendering without an explicit architectural decision.

### Component Rules

- Use existing shadcn/ui primitives (`src/components/ui/`) before creating new base components.
- Do not copy-paste and rename shadcn components — extend via props or wrapper components.
- All forms must use React Hook Form + Zod. Do not use uncontrolled forms or custom validation logic.
- User-generated content (reviews, names, business descriptions) must pass through `src/lib/sanitize.ts` before rendering as HTML. Never use `dangerouslySetInnerHTML` on unsanitized user input.
- RTL layout: use Tailwind `rtl:` variants. Do not hardcode `left`/`right` positioning without RTL consideration. The app is `dir="rtl"` at the HTML level.

### Downstream Impact

- Before changing a shared hook, context, or utility function, search for all usages (`Grep`) and trace the effect on each consumer.
- Before modifying an Edge Function, identify all callers: frontend pages, other Edge Functions, cron triggers, webhooks.
- Before adding a DB column or changing a column type, plan the migration and the types regeneration together.
- Before changing a shared component (a button, card, or modal used across many pages), check all pages that render it.

---

## Debugging Discipline

1. Reproduce the bug in a specific, named user scenario before proposing a fix.
2. Identify the exact failure point: component render, React Query fetch, hook logic, Edge Function, RPC, or RLS policy.
3. Distinguish data bugs from logic bugs: inspect what Supabase returns before fixing the code that processes it.
4. Verify the fix by tracing the same scenario from start to finish after the change.
5. Do not mark an issue resolved based on static code review alone.

---

## False Completeness Signals

These phrases indicate incomplete work and require verification before closing a task:

- "This should work" → verify it does
- "I've updated the component" → test the component in its actual page context
- "The migration has been applied" → confirm via migration list or Supabase dashboard
- "SEO is improved" → confirm meta tags actually appear in the served HTML
- "Auth is fixed" → test the happy path AND all error states
- "Trust score now reflects X" → verify the score computation query and its inputs

---

## Environment Variables Reference

| Variable | Scope | Purpose |
|---|---|---|
| `VITE_SUPABASE_URL` | Frontend | Supabase project URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Frontend | Supabase anon/publishable key |
| `RESEND_API_KEY` | Edge Functions only | Transactional email |
| `TURNSTILE_SECRET_KEY` | Edge Functions only | Cloudflare Turnstile verification |
| `CRON_SECRET` | Edge Functions only | Cron job auth header |
| `FRONTEND_URL` | Edge Functions only | Public app URL for links/redirects |

Frontend code cannot and must not access `RESEND_API_KEY`, `TURNSTILE_SECRET_KEY`, or any other server-side secret.

---

## Key Files Quick Reference

| File | Purpose |
|---|---|
| `src/App.tsx` | All React Router routes |
| `src/contexts/AuthContext.tsx` | Auth, MFA, rate limiting, session timeout, subscription state |
| `src/contexts/ModeContext.tsx` | User/Business mode toggle (localStorage) |
| `src/integrations/supabase/client.ts` | Supabase JS client (sessionStorage, PKCE) |
| `src/integrations/supabase/types.ts` | Auto-generated DB types — never hand-edit |
| `src/hooks/useFeatureGating.ts` | Subscription tier → feature access mapping (free/pro/enterprise) |
| `src/lib/auth-security.ts` | `ClientRateLimiter`, `SessionTimeout` implementations |
| `src/lib/sanitize.ts` | XSS prevention for user-generated content |
| `src/lib/affiliate.ts` | Referral/affiliate tracking helpers |
| `src/components/BusinessHero.tsx` | Business profile hero — trust badges, affiliate CTA, contact |
| `src/components/BusinessCard.tsx` | Business listing card — used in search/homepage grids |
| `src/components/ReviewCard.tsx` | Review display with verification badges, proof indicators |
| `src/components/ServiceCard.tsx` | Service/product card with dynamic affiliate CTA |
| `src/components/CouponInput.tsx` | Coupon code input UI (used in settings page) |
| `src/components/AccessibilityMenu.tsx` | Accessibility toolbar (9 a11y features, persisted in localStorage) |
| `src/pages/business/PricingPage.tsx` | Pricing plans (coupon-based, no active payment provider) |
| `src/pages/business/BusinessSettingsPage.tsx` | Business settings — profile, contact, affiliate URL, coupon activation |
| `supabase/functions/` | 39 Deno TypeScript Edge Functions |
| `supabase/functions/apply-coupon/` | Coupon redemption (supports lifetime via duration_months=0) |
| `supabase/functions/check-subscription/` | Subscription state check (handles lifetime NULL expiry) |
| `supabase/migrations/` | 78+ SQL migration files (applied history) |
| `public/_headers` | Netlify security headers (CSP, HSTS, MIME) |
| `vite.config.ts` | Build config + CSP header configuration |
| `SYSTEM_COHERENCE.md` | Trust/Points architecture design doc (reference only) |

## Review Verification Tiers

Reviews have 4 trust tiers that determine how they affect the Trust Score:

| Tier | Label | Source | Counts in Trust Score |
|---|---|---|---|
| T1 | ביקורות מאומתות | Verified purchase proof | Yes |
| T2 | WhatsApp / Instagram | Business-approved feedback | Partial |
| T3 | Google / Facebook | Imported external reviews | No |
| T4 | קהילה | Community (no purchase proof) | No |

Only T1 reviews count toward the Trust Score. All others are displayed separately. The `ReviewSourceBreakdown` component shows this breakdown on business profiles. Never claim "all reviews are verified" — use "ביקורות מאומתות" for T1 and acknowledge community reviews exist.

## Accessibility System

The accessibility menu (`AccessibilityMenu.tsx`) toggles CSS classes on `<html>`:
- `a11y-high-contrast`, `a11y-reduced-motion`, `a11y-link-highlight`, `a11y-readable-font`, `a11y-big-cursor`, `a11y-grayscale`, `a11y-text-spacing`, `a11y-invert-colors`

The CSS for these classes is in `src/index.css`. The `a11y-invert-colors` class includes extensive overrides for hardcoded Tailwind dark classes (`text-white`, `bg-zinc-*`, etc.) — if you add new hardcoded dark colors, you must also add overrides in the invert-colors section.

## Storage Buckets & RLS

Upload paths must use `auth.uid()` as the first folder segment to match storage RLS policies:
- `covers` bucket: path = `{userId}/cover.ext`
- `avatars` bucket: path = `{userId}/avatar.ext`
- `testimonials` bucket: path = `{businessId}/filename.ext` (RLS checks business ownership)

Using `businessId` instead of `userId` in covers/avatars will cause "row-level security policy" errors.

---
> Source: [elior1324/reviewwise-hub](https://github.com/elior1324/reviewwise-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
