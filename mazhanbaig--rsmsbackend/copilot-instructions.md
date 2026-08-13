## rsmsbackend

> You are working on BackendRSMS, the backend for a real estate management SaaS

# BackendRSMS — Project Context (read this before doing anything)

You are working on BackendRSMS, the backend for a real estate management SaaS
(Z-State/RSMS). This file is your persistent memory across sessions — read it fully
before starting any task.

## NON-NEGOTIABLE RULES

1. **Never merge `dev` into `main`.** All work happens on `dev` only, unless
   explicitly told otherwise in a specific task. This has been violated once before by
   a prior agent — do not repeat that. If you're ever unsure, ask rather than merge.
2. **Simple, boring code over clever code.** Every choice should still be secure,
   performant, and reliable — but prefer the straightforward standard approach over a
   custom abstraction. A junior developer should be able to read any file and
   understand it without explanation.
3. **Never mark something "done" without real verification evidence** — actual
   command output (curl responses, test results, row counts), not a description of
    what should happen. Log every task's start and result in `docs/archive/REPORT.md`,
    appending to the existing file, not overwriting it.
4. **Commit messages matter** — this repo is meant to look professional to anyone
   browsing it. Format: `type(scope): summary` + a 2-4 line body explaining why, not
   just what. Types: feat, fix, refactor, chore, docs, test, security.
5. **Scope discipline** — if a task is scoped to X, do X. If you notice something else
    worth fixing along the way, note it in docs/archive/REPORT.md as a follow-up, don't fix it
   inline in the same change. Mixing concerns makes it impossible to isolate what
   broke if something regresses.
6. **Payments stay disabled** — `PAYMENTS_ENABLED=false` in both frontend and backend
   env vars. Do not change this without explicit instruction. Full JazzCash + Easypaisa
   logic exists and is built, it's just gated off intentionally.

## CURRENT ARCHITECTURE

- **Stack:** Express.js v5, layered as routes → controllers → services → middlewares.
- **Auth:** Firebase Authentication (untouched, unchanged, not part of any migration).
  `req.user.uid` is the verified Firebase identity on every protected route.
- **Database:** Postgres via Neon, accessed through Prisma ORM. This REPLACED
  Firebase Realtime Database for all entity data (clients, owners, properties,
  events, tasks, transactions). Firebase RTDB still exists as a read-only rollback
  safety net — do not write new code that depends on it going forward unless a task
  specifically says so.
- **Schema (8 Prisma models):** Organization → User → { Client, Owner, Property,
  Event, Task, Transaction }, all foreign-key related, every entity scoped by
  `userId` for ownership isolation (a user must NEVER be able to read/write another
  user's records — this is enforced via `where: { id, userId }` together on every
  lookup, never `id` alone).
- **API shape:** Per-entity REST routes (`/api/clients`, `/api/owners`,
  `/api/properties`, `/api/events`, `/api/tasks` — each with GET list, POST, GET/:id,
  PUT/:id, DELETE/:id). The old generic `/api/data?path=...` endpoint has been
  removed (recoverable from git history at commit `97e7ab3` if ever needed).
- **Payments:** JazzCash + Easypaisa logic fully built (payment creation, webhooks,
  HMAC verification, transaction persistence to Postgres) but gated behind
  `PAYMENTS_ENABLED=false` on both frontend and backend.
- **Security hardening already done:** Helmet, pino-http logging, Upstash Redis rate
  limiting (falls back to in-memory with a warning if Redis env vars aren't set),
  1MB body size limits, firebase-admin upgraded to v14 (modular API), Sentry error
  monitoring wired into all controllers.
- **Testing:** Jest service-layer tests exist for all 5 entities + auth (49 tests as
  of the last verification pass), specifically proving ownership isolation. Run
  `npm test` before considering any service-layer change complete.

## DOCUMENTATION LAYOUT

- `AGENTS.md` (this file) — read every session, kept at repo root.
- `FRONTEND_MIGRATION_GUIDE.md` — at repo root; read before any frontend-facing
  backend work (documents the old `/api/data` → per-entity REST migration).
- `docs/ENVKEYS.md` — full env-var reference with where-to-find-each-key steps.
- `docs/PAYMENTS_ENABLEMENT_GUIDE.md` — steps to safely enable payments (gated off).
- `docs/archive/` — historical session reports (REPORT.md full log,
  COMPREHENSIVE_AUDIT.md, MASTER_FINAL_CHECK.md, BACKEND_AUDIT_1.md). Preserved for
  record; NOT loaded by default. Consult when investigating past decisions/regressions.

## KNOWN OPEN ITEMS (check docs/archive/REPORT.md for full detail before assuming these are stale)

- Dual-write to Firebase RTDB still happens in `authService.js` and
  `paymentService.js` (not the 5 entity services) — flagged for eventual removal
  after a production stability window, not yet scheduled.
- 2 client records + 2 property records in Firebase RTDB were skipped during backfill
  (orphaned — no matching Postgres User existed at backfill time).
- Events and Tasks have no cross-entity references (`clientId`/`propertyId`) populated
  from the backfill, since the original Firebase records never stored those as flat
  fields.
- `main` and `dev` branches diverged slightly after a REPORT.md-only commit landed on
  `main` directly — check `git log --oneline --graph --all` before assuming which
  branch has the latest state.

## KNOWN GOTCHAS

- **ESM-only transitive dependencies (CommonJS backend):** This backend uses
  CommonJS (`require()`). Any dependency that pulls in an ESM-only package will
  crash the entire serverless function at import time (`ERR_REQUIRE_ESM`). Before
  adding a new npm dependency, check its transitive deps for `"type": "module"` in
  their `package.json` — especially for modern/crypto-related packages. This exact
  failure mode (works locally in dev with looser module resolution, crashes in
  Vercel's stricter serverless bundling) is easy to miss until it's already in
  production. If a dependency introduces ESM-only transitive deps, pin to an older
  version that avoids them, or replace with a CJS-safe alternative.
  **Known pinned packages (do not remove these overrides without checking the ESM chain):**
  - `otplib` pinned to exact `13.0.1` in dependencies — v13.1+ pulls `@scure/base@^2` (ESM-only)
  - `jwks-rsa` pinned to `3.2.2` via `overrides` — v4.x pulls `jose@^6` (ESM-only); firebase-admin depends on jwks-rsa

## WORKFLOW FOR EVERY TASK

1. Read the specific task instructions given to you.
2. Check docs/archive/REPORT.md and this file for relevant existing context before
   starting — don't rebuild something that already exists.
3. Log what you're about to do in docs/archive/REPORT.md before starting.
4. Do the work, scoped exactly to what was asked.
5. Verify with real command output — paste it into REPORT.md, not a summary.
6. Commit to `dev` with a properly formatted message.
7. Push to `dev`. Never touch `main`.
8. Report back in plain language what's done, what's remaining, and flag any
   decision that needs human input rather than guessing.

---
> Source: [mazhanbaig/RSMSBackend](https://github.com/mazhanbaig/RSMSBackend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
