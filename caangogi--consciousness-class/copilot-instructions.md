## consciousness-class

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Consciousness Class — a Next.js 15 (App Router) platform for holistic creators / therapists / coaches. The product is a unified "Asset Catalog" with six sellable asset types: **Course, Membership, Coaching, Podcast, Community, Download**. Users have one of four roles: `student`, `creator`, `admin`, `superadmin`.

The active roadmap and architectural state-of-affairs lives in [docs/hoja-de-ruta.md](docs/hoja-de-ruta.md). Brand/style intent (Natureza palette, Apple HIG, `ios-list` patterns) is in [docs/blueprint.md](docs/blueprint.md). Read both before making product-shaped decisions.

## Commands

```bash
npm run dev          # next dev --turbopack on port 9003
npm run build        # next build
npm run start        # next start
npm run lint         # next lint
npm run typecheck    # tsc --noEmit (the source of truth — see "Build vs typecheck" below)

npm test             # vitest run (unit + integration, single pass)
npm run test:watch   # vitest --watch (development)
npm run test:ui      # vitest --ui (visual debug in browser)
npm run test:e2e     # playwright test (auto-starts npm run dev on :9003)
npm run test:e2e:ui  # playwright UI mode for debugging
npm run test:e2e:headed  # see the browser while tests run

npm run genkit:dev   # Genkit dev server, entry src/ai/dev.ts
npm run genkit:watch # Genkit dev server with --watch
```

Utility scripts (run with `npx tsx` or `node`):
- [scripts/seed-db.ts](scripts/seed-db.ts) — seeds Firestore (loads `.env.local` then `.env`).
- [scripts/set-superadmin.js](scripts/set-superadmin.js) — promotes a user to `superadmin` in Firestore.

## Testing strategy (READ BEFORE CODING)

Full philosophy in [documentation/testing-strategy.md](documentation/testing-strategy.md). Three modes by domain:

| Mode | Where it applies |
|------|-----------------|
| **TDD strict** (test-first, separate `test(...)` commit before `feat(...)`) | `src/backend/payments/`, `src/backend/wallet/`, `src/backend/referrals/`, `src/backend/booking/` (state machine + refund rules), `src/app/api/webhooks/` |
| **Test-after rigorous** (test before merge, ≥70% module coverage) | API routes (wire-up), services, helpers, utilities |
| **Eval-first** (golden set + human review, no `expect()` against LLM output) | Everything in `src/ai/` and AI endpoints |

**Critical rule for TDD-strict directories:** the PR must show the failing `test(...)` commit *before* the `feat(...)` commit that makes it pass. If review can't see that sequence in git history, the PR is rejected.

When you (Claude Code) are about to edit a file in a TDD-strict directory, **stop and write the failing test first**. Commit it. Then write the implementation. Commit that separately.

### E2E (Playwright)

E2E tests live in `e2e/` (NOT mixed with unit tests in `src/`). Config in [playwright.config.ts](playwright.config.ts) auto-starts `npm run dev` on port 9003. First-time setup on a new machine: `npx playwright install chromium`.

The current smoke test ([e2e/smoke.spec.ts](e2e/smoke.spec.ts)) verifies that the public surfaces (home, /products, /courses redirect) render without 5xx. Behavioral E2E (full checkout, booking, RAG companion) lands per-feature in fases 5/6 once seedable Firebase + Stripe test data exists.

## Build vs typecheck

`next.config.ts` sets `typescript.ignoreBuildErrors: true` and `eslint.ignoreDuringBuilds: true`. **`npm run build` will pass even with type errors.** Use `npm run typecheck` to verify TS correctness — never trust a green `build` alone. ([ts_errors.txt](ts_errors.txt) at the repo root is a historical dump from one such run, not a live artifact.)

## Architecture

### Backend: hexagonal per asset type

Each business domain under [src/backend/](src/backend/) follows the same three-layer structure:

```
src/backend/<domain>/
  domain/          entities/, repositories/      (pure interfaces + entity classes)
  application/     <name>.service.ts             (use cases — orchestrates repos + Stripe + catalog)
  infrastructure/  dto/, repositories/           (Firebase repo impls, request/response DTOs)
```

Domains: `course`, `membership`, `coaching`, `community`, `podcast`, `download`, `booking`, `catalog`, `enrollment`, `progress`, `referral`/`referrals`, `finance`, `payments`, `user`, `shared`. Wiring goes **API route → Service → Repository interface → Firebase impl**; never have a route talk to Firestore directly.

### Catalog unification

Whenever a Course/Membership/Coaching/Podcast/Community/Download asset is created or updated, its service must sync a lightweight `CatalogItem` financial record via `CatalogService` (see the pattern in [src/backend/course/application/course.service.ts](src/backend/course/application/course.service.ts)). The unified storefront and creator catalog (`/api/creator/catalog`, `/dashboard/products`) read from this catalog, not from per-domain collections. When adding a new asset type, replicate the catalog sync — features that skip it become invisible to checkout.

### Enrollments

The legacy `users.cursosInscritos` array has been replaced by a universal `inscripciones` subcollection backed by `EnrollmentEntity` (handles all six asset types). New access checks should query `inscripciones`, not the array. The `cursosInscritos` field still exists in `UserProfile` for backwards-compat reads — don't add new writers to it.

### Frontend (Next.js App Router)

- `src/app/(main)/` — public marketing-style routes (group, no URL prefix).
- `src/app/dashboard/` — single dashboard shell (layout.tsx) for *all* authenticated roles. Sub-routes (`builder/`, `products/`, `community/`, `finances/`, `learning/`, `users/`, `availability/`, `settings/`) are gated by role inside the components, not by route. Older `dashboard/creator/`, `dashboard/student/`, `dashboard/superadmin/` directories were collapsed into this unified shell — `git status` will show them as deleted.
- `src/app/api/` — REST endpoints, grouped by audience (`creator/`, `student/`, `superadmin/`, `learn/`, `checkout/`, `webhooks/stripe/`, etc.).
- `src/app/courses/` and `src/app/products/` — public detail/listing pages. `next.config.ts` 301-redirects `/courses` → `/products`; treat `/products` as canonical.

Path alias: `@/*` → `src/*` (see [tsconfig.json](tsconfig.json)).

### Auth & RBAC

- Client: [src/contexts/AuthContext.tsx](src/contexts/AuthContext.tsx) wraps Firebase Auth + a Firestore `users/{uid}` profile read. The `caangogi@gmail.com` email is hardcoded as a dev-superadmin override.
- Server: [src/lib/auth/rbac.ts](src/lib/auth/rbac.ts) exposes `requireRoles(request, allowedRoles)` which verifies the `Authorization: Bearer <idToken>` header via `adminAuth.verifyIdToken` and reads the role from the Firebase custom claim `role` (defaulting to `student`). The same dev-superadmin email override applies server-side. Every protected API route should start with `requireRoles` — follow the existing pattern (early-return the `error` response).

### Firebase Admin

[src/lib/firebase/admin.ts](src/lib/firebase/admin.ts) is highly defensive about `FIREBASE_ADMIN_PRIVATE_KEY`: it trims whitespace, strips surrounding quotes, and replaces literal `\n` with newlines. If you add a new env-loading code path for the admin SDK, do not bypass that key normalization. Required env vars: `FIREBASE_ADMIN_PROJECT_ID`, `FIREBASE_ADMIN_CLIENT_EMAIL`, `FIREBASE_ADMIN_PRIVATE_KEY`, `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`. Stripe code is wrapped in `if (process.env.STRIPE_SECRET_KEY)` and degrades gracefully when unset — keep that pattern when adding Stripe calls so the app still boots locally without a secret.

### AI (Genkit + Gemini)

- [src/ai/genkit.ts](src/ai/genkit.ts) configures Genkit with the `googleAI()` plugin and default model `googleai/gemini-2.0-flash`.
- [src/ai/dev.ts](src/ai/dev.ts) is the entry for `npm run genkit:dev`.
- AI features in product copy (Assistive Markdown Editor, AI cover generation a.k.a. "Nano Banana", Magic Document, Coaching session plan, Community guidelines) typically use `gemini-2.5-flash` and live behind `/api/creator/assets/ai-cover` and similar endpoints — check existing endpoints before adding new ones; the team prefers a single generic AI endpoint over per-feature ones.

### Stripe

Subscription products and one-shot prices are managed by services (see `manageStripeProductAndPrice` in `course.service.ts`). The **canonical webhook handler** is at [src/app/api/webhooks/stripe/route.ts](src/app/api/webhooks/stripe/route.ts) — handles checkout, subscription create/update/delete, and invoice events; has the F1.4b idempotency guard via `ProcessedStripeEventEntity`. Checkout sessions are created at `src/app/api/checkout/create-session` and `create-subscription-session`.

> **⚠️ TWO webhook handlers exist on purpose** — see [documentation/decisions/0001-defer-wallet-ledger-migration.md](documentation/decisions/0001-defer-wallet-ledger-migration.md). The second one at `src/app/api/checkout/webhook/route.ts` implements the target ledger architecture (writes to `wallets` collection via `WalletTransactionEntity`) but only handles `checkout.session.completed`. **Do not delete it** — it's the migration target, not dead code. Stripe Dashboard currently points only at `/api/webhooks/stripe` (the principal). Until the dedicated "Wallet Ledger Migration" sprint runs, route all new event handling through the principal and treat `wallets` as a sleeping store.

### Wallet

[src/backend/wallet/](src/backend/wallet/) is the dedicated module for balances (extracted from `finance/` in T0.1 per PM decision D1). Today the principal webhook still mutates `usuarios.balance*` directly — wallet ledger transactions exist but are only written via the dormant secondary webhook. The migration that flips reads/writes to the ledger is scheduled for a future sprint (see ADR-0001).

## Logging

- New code MUST use the structured logger at [src/lib/logger.ts](src/lib/logger.ts) (`import { logger } from '@/lib/logger'`). It emits Cloud Logging-compatible single-line JSON in production and pretty text in dev.
- Levels: `logger.debug`, `logger.info`, `logger.warn`, `logger.error`. Each takes `(message: string, context?: Record<string, unknown>)`.
- `Error` instances passed in context are auto-serialized (name + message + stack + cause).
- Pre-existing `console.*` calls (especially in `src/app/api/webhooks/stripe/route.ts`) will be migrated incrementally — don't sweep them in unrelated PRs.

## UI patterns

### Brand mark

`<Logo />` ([src/components/shared/Logo.tsx](src/components/shared/Logo.tsx)) is the single source of truth for the brand block. It renders a `Leaf` glyph (Pluma) in a terracotta-tinted rounded square + the "Consciousness Class" wordmark. **Do not fetch logo PNGs from Firebase Storage** — that path was retired because (a) the bucket 402'd in production when billing wasn't active, and (b) an inline SVG avoids CLS and respects `currentColor` in dark mode. Use `<Logo iconOnly />` in compact headers and the `size` prop (`sm | md | lg`) to scale.

### Empty states

When a list/table has zero rows, **use `<EmptyState />`** from [src/components/shared/EmptyState.tsx](src/components/shared/EmptyState.tsx). It centralizes the dashed-border card + tinted icon square + headline + body + pill CTA pattern. Example:

```tsx
import { EmptyState } from '@/components/shared/EmptyState';
import { BookOpen } from 'lucide-react';

<EmptyState
  icon={BookOpen}
  tint="chambray"
  title="Tu librería está vacía"
  description="Cuando compres un curso, te unas a una membresía o reserves una sesión de coaching, aparecerá aquí."
  primary={{ label: 'Explorar el catálogo', href: '/products' }}
/>
```

Tints available: `chambray | terracotta | sage | olive | primary | muted`. Use `dense` for tighter contexts (inside cards). Use a bespoke component instead of `<EmptyState />` only when the empty state needs extra content (e.g. `/dashboard/products` shows a 6-asset teaser grid) or when it's actually an error/filter-clear state — those have different affordances.

When writing the copy:
- **Title** is one sentence, no period. Describes the current state from the user's perspective ("Sin movimientos aún", not "No transactions found").
- **Description** explains *what would populate this surface* and *why it's currently empty*. Avoids dead-end phrasing — invites the next action.
- **Primary CTA** is the highest-intent action. If you can't think of one, omit it (no CTA is better than a weak CTA).

## Conventions worth knowing

- The codebase mixes Spanish (domain language: `nombre`, `precio`, `tipoAcceso`, `cursosInscritos`, `inscripciones`) and English (technical terms). Keep new domain fields in Spanish to match — consistency matters more than language preference.
- The dashboard is reached at `/dashboard`; UI follows Apple HIG with `ios-list` patterns and the Natureza palette (Olive/Sage primary, Bone background, Charcoal foreground, Terracotta accent). New surfaces should match.
- `react-beautiful-dnd` is used for drag-and-drop module/lesson reordering.
- `next.config.ts` ignores TS and ESLint errors during build. Always run `npm run typecheck` before declaring a task done.

## Files to ignore

Stray scripts at repo root (`patch.js`, `patch_manager.js`, `patch_manager_final.js`, `patch_new.js`, `list_models.js`, `models.txt`, `test-db.js`, `ts_errors.txt`, `tsconfig.tsbuildinfo`) are leftover tooling/artifacts. Don't treat them as project entry points; don't refactor them unless asked.

---
> Source: [caangogi/consciousness-class](https://github.com/caangogi/consciousness-class) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
