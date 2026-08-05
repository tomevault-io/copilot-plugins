## kirim-saas

> Modular, type-safe SaaS starter. Bun monorepo, Hono API, Vite/React web, Drizzle + Postgres, Better Auth, provider-agnostic payments, hardened by default (OWASP audit passing).

# saas-boilerplate

Modular, type-safe SaaS starter. Bun monorepo, Hono API, Vite/React web, Drizzle + Postgres, Better Auth, provider-agnostic payments, hardened by default (OWASP audit passing).

Read this file first. Then read the AGENTS.md next to whatever you're about to edit — each package documents its local invariants.

## Repository map

```
saas-boilerplate/
├── apps/
│   ├── api/          Hono + zod-openapi + Better Auth handler (Bun runtime)
│   ├── web/          Vite + React 19 + shadcn/ui + TanStack Router + i18next
│   └── worker/       Bun + BullMQ background jobs (billing reconciler, subscription lifecycle)
├── packages/
│   ├── config/       tsconfig presets + Zod env schema
│   ├── shared/       Enums, AppError, Result, crypto helpers — zero I/O, safe everywhere
│   ├── db/           Drizzle schema, client, migrations, seed
│   ├── auth/         Better Auth server factory + browser client
│   ├── email/        Resend mailer + React Email templates
│   ├── payments/     Provider interface + Duitku adapter (Midtrans/Xendit/Stripe are interface-only stubs that throw until implemented)
│   └── storage/      Provider interface + R2/S3 adapter — presigned uploads, magic-byte verify
├── docker-compose.yml    Postgres + Redis + Mailpit for local dev
├── Dockerfile.api        Multi-stage Bun runtime (non-root)
├── Dockerfile.web        Multi-stage nginx-unprivileged (port 8080)
├── renovate.json         Grouped dependency PRs, auto-merge safe minors
└── .github/workflows/    CI (lint/typecheck/test + e2e) and Docker publish to GHCR
```

## Golden rules

1. **No circular deps.** `apps/*` depends on `packages/*`, never the reverse. `packages/db` and `packages/shared` are leaf packages with no cross-package runtime deps.
2. **No I/O at import time.** Every package exports pure factories (`createXxx(config)`). The API's `server.ts` is the ONE place that reads env and constructs services.
3. **Money is integer minor units.** `priceAmount`, `amount` are integers. Currency is a separate column. No floats, ever.
4. **Payments are provider-agnostic.** Routes talk only to the `PaymentProvider` interface. Adding a provider = one file + one factory branch. Swapping providers = one env var.
5. **Errors are `AppError` at package boundaries.** The API error middleware is the ONLY response formatter. Frontend surfaces them via `normalizeError()` + i18n code table.
6. **Type-safe env.** `@saas/config/env` validates once at boot and throws on missing/invalid vars. Placeholder values in `.env.example` are actively rejected.
7. **OpenAPI is source of truth for the HTTP contract.** Every route uses `createRoute(...)`. `/api/openapi.json` and `/api/docs` are gated OFF in production.
8. **i18n covers every user-visible string.** Locales are split per namespace under `apps/web/src/i18n/locales/{en,id}/<ns>.ts`. Add keys to BOTH `en/<ns>.ts` and `id/<ns>.ts` in the same commit. The `id` aggregator is typed against `typeof en`, so a missing key fails typecheck instead of silently falling back. Error codes are looked up via `errors.<CODE>` in `errors.ts`.
9. **Column-level encryption for secrets at rest.** Two independent mechanisms, do NOT conflate them: (a) **OAuth tokens** on `accounts.*` are encrypted by Better Auth's built-in `account.encryptOAuthTokens` (its own ciphertext format, keyed off `AUTH_SECRET`) — NOT by `@saas/shared/crypto`, whose format Better Auth's OAuth refresh path cannot read. (b) **Verification tokens** (`verifications.value`) and any third-party integration secrets go through `@saas/shared/crypto` (AES-256-GCM), gated on `COLUMN_ENCRYPTION_KEY` — when that key is unset the hook is a no-op and values are stored plaintext (dev default; production MUST set it — the env schema throws at boot without it). See `packages/auth/AGENTS.md` "Column-level encryption". Personal access tokens (`api_keys.hashed_key`) use `hashApiKey` from `@saas/shared/crypto`, which returns a raw 32-byte `Buffer` (sha256 digest) mapped to a `bytea` column — NOT hex. Plaintext is returned to the caller ONCE on create and NEVER persisted or logged.
10. **Webhooks: verify signature, re-check business fields, transact.** Never trust that a signed payload is safe to write — cross-check amount/currency against the pending row. Persist raw payloads ONLY through the adapter's `redactWebhookPayload()` — the DB is not a place for provider signatures, customer PII, or free-form fields. Every `PaymentProvider` MUST implement `redactWebhookPayload(raw)`; per-provider allowlists are the invariant. Redacted payloads land in the sibling table `payment_webhook_events` (append-only), NOT on the `payments` row — that keeps the hot `/billing/payments` history query narrow. See `packages/payments/AGENTS.md`.
11. **Tenant scoping is a security invariant, not a filter.** Every read/write on business data MUST include `WHERE organization_id = ?`. The active org is resolved via `requireActiveOrg(services, session)` — never trust the client. Missing the predicate is a cross-tenant data leak, not a bug.
12. **This is a tenant application, not a platform-admin surface.** Roles (`owner | admin | member`) are per-organization. There is no "platform admin". Dashboard stats, MRR, revenue — all scoped to ONE workspace. Do not add cross-tenant aggregations to `/api/*`; that belongs in a separate admin surface not shipped here.
13. **Bearer auth inherits creator's role at request time + carries hierarchical scopes.** API keys grant the same tenant role their creator had when the request runs. Each key ALSO carries an explicit scope array (`api_keys.scopes`, `jsonb` — see `API_KEY_SCOPES` in `@saas/shared/constants`). Default is `['read']`; `write` and `admin` are opt-in at create time. Scopes are hierarchical: `admin` implies `write` implies `read` (via `SCOPE_HIERARCHY` + `scopeImplies`). Bearer mutation endpoints call `requireScope(c, 'write')` from `apps/api/src/lib/scopes.ts` to reject read-only keys BEFORE the role check; org-wide destructive endpoints (billing cancel, workspace delete) call `requireScope(c, 'admin')`. Cookie sessions bypass scope entirely (they carry the full role). **Elevation guard**: a bearer caller can only mint keys with scopes THEY currently hold — a `write`-scoped key cannot bootstrap an `admin`-scoped child key (enforced in `POST /api/api-keys`). Built-in identity flows (password change, account deletion, email change) live under Better Auth's `/api/auth/*`, where bearer sessions are invisible (Better Auth resolves cookies itself) — no extra guard needed. Any CUSTOM sensitive route you add outside Better Auth MUST call `requireCookieAuth(c)` from `apps/api/src/lib/session-source.ts` to reject bearer callers regardless of scope (currently zero call sites, by design — see the helper's doc comment for the pattern). Bearer sessions get a random opaque `session.id` + `session.token` per request — NEVER log them (they'd become stable identifiers tying a request back to a specific `api_keys` row). See `apps/api/AGENTS.md` "Bearer auth".

## Tech stack (locked versions)

| Layer          | Choice                                                      |
|----------------|-------------------------------------------------------------|
| Runtime        | Bun 1.3 (also engines constraint in root `package.json`)    |
| API framework  | Hono 4.12 + `@hono/zod-openapi` 1.4 + `@hono/swagger-ui`    |
| Auth           | Better Auth 1.6 (organization plugin, multi-tenant)         |
| DB             | Postgres 16 + Drizzle 0.45 + `postgres.js` 3.4              |
| Web framework  | Vite 8.1 + React 19.2                                       |
| Web routing    | TanStack Router 1.170 (file-based, plugin 1.168)            |
| Server state   | TanStack Query 5.101                                        |
| UI             | shadcn/ui (new-york style) + Tailwind 4.3 + Radix           |
| Icons          | lucide-react 1.24 (no brand icons — use inline SVG for Github) |
| Fonts          | Self-hosted JetBrains Mono via `@fontsource-variable/*`     |
| i18n           | i18next 26 + react-i18next 17                               |
| Email          | Resend 6 + React Email 6                                     |
| Payments       | Duitku (implemented) · Midtrans/Xendit/Stripe (interface only) |
| Validation     | Zod 4.4 (root `overrides` pins to a single version)         |
| Lint/format    | Biome 2.5                                                    |
| Test           | Vitest 3.2 (unit) + Playwright 1.61 (e2e) + `bun test`      |
| Toast          | sonner (themed via `components/ui/sonner.tsx`)              |
| CI             | GitHub Actions                                               |
| Deploy         | Generic Docker (VPS, Fly.io, Railway) images pushed to GHCR |
| Deps           | Renovate config: grouped PRs, digest pinning                |

## Quick start

```bash
# 1. Install
bun install

# 2. Boot Postgres + Mailpit
bun run docker:up

# 3. Env
cp .env.example .env
# Generate AUTH_SECRET:
#   openssl rand -hex 32
# Optionally add DUITKU_* to test payments end-to-end.

# 4. Migrate + seed
bun run db:generate    # after any schema change
bun run db:migrate
bun run db:seed

# 5. Run
bun run dev            # runs api + web in parallel
# → API   http://localhost:3000
# → Docs  http://localhost:3000/api/docs   (dev only)
# → Web   http://localhost:5173
# → Mail  http://localhost:8025
```

Ports are load-bearing. Playwright, CORS, and Better Auth's `trustedOrigins` all assume them.

## Common tasks

```bash
bun run lint            # biome check .
bun run lint:fix        # biome check --write . (auto organizes imports too)
bun run typecheck       # all packages
bun run test            # bun test packages/ + vitest apps/web/src
bun run test:e2e        # playwright
bun run db:studio       # drizzle-kit studio
```

## Adding a feature — decision tree

- **New DB table** → `packages/db/src/schema/<domain>.ts`, re-export from `index.ts`, `bun run db:generate`, commit the generated SQL under `drizzle/`.
- **New notification/audit event** → don't invent a new table. Use `notifications` (in-app messages to a user in a workspace) or `audit_logs` (append-only "who did what") — emitters live near their action (e.g. `apps/api/src/routes/billing.ts` calls `recordAudit` after cancel). See `apps/api/AGENTS.md` "Audit events".
- **New API route** → `apps/api/src/routes/<name>.ts` with `createRoute(...)`. Register in `app.ts`. Auth-required routes use `requireSession`; write endpoints use `requireRole(services, session, 'admin')`.
- **New page** → `apps/web/src/routes/<path>.tsx` with `createFileRoute`. Add i18n keys to the relevant namespace file in `apps/web/src/i18n/locales/en/<ns>.ts` AND `id/<ns>.ts` — need a new namespace? See `apps/web/AGENTS.md`. **Do NOT wrap dashboard pages in `<AppShell>` or subtree sub-pages in `<SectionLayout>`** — those are mounted by `app.tsx`, `app.account.tsx`, and `app.workspace.tsx` respectively. Child pages render `<PageHeader />` + content. Marketing/public pages (outside `/app/*`) DO wrap themselves in `<MarketingShell>`. See `apps/web/AGENTS.md` "Layout composition" for the full ownership table and correct/wrong templates.
- **New account setting** (personal, user-scoped: profile fields, MFA, personal tokens, personal notifications) → `apps/web/src/routes/app.account.<name>.tsx`. Add an entry to `apps/web/src/components/app/account-nav-config.ts` and an i18n key under `account.nav.<name>` in both `en/account.ts` and `id/account.ts`. No `minRole` gating — every user has full control over their own account.
- **New workspace setting** (org-scoped: members, billing, org settings, audit log) → `apps/web/src/routes/app.workspace.<name>.tsx`. Add an entry to `apps/web/src/components/app/workspace-nav-config.ts` (set `minRole: 'admin'` if only admins should see the tab) and an i18n key under `workspace.nav.<name>` in both `en/workspace.ts` and `id/workspace.ts`. Also gate the page component with `<RequireRole min="admin">` and the corresponding API endpoint with `requireRole(services, session, 'admin')` — the nav `minRole` is a UI hint only.
- **New reusable UI pattern** → check `apps/web/src/components/{empty-state,confirm-dialog,price-display,role-badge,copy-button,date-display,data-table,text-field}.tsx` first. Adding a new one belongs in `apps/web/AGENTS.md` "Shared UI primitives" — mirror the existing shadcn v4 style (function comp, `data-slot`, no `forwardRef`).
- **New payment provider** → new file under `packages/payments/src/providers/`, branch in `factory.ts`, tests for signature verification. See `packages/payments/AGENTS.md`.
- **New storage provider** → new file under `packages/storage/src/providers/`, branch in `factory.ts`, tests for signed URL structure. See `packages/storage/AGENTS.md`.
- **File upload from a new place** → call `POST /api/files/upload-url` with a `scope` that is already in `FILE_SCOPES` (`avatar`, `org-logo`, `attachment:*`). Adding a new scope = one entry in `@saas/shared/constants.FILE_SCOPES` + a route-level role check.
- **New email** → `packages/email/templates/YourEmail.tsx`, re-export, call `mailer.send({ react: <YourEmail {...props} /> })`.
- **New OAuth provider** → extend `AuthConfig` and `socialProviders` in `packages/auth/src/server.ts`. Add env vars to `@saas/config/env` AND `.env.example`.
- **New error code** → add key to `errors.<CODE>` in `apps/web/src/i18n/locales/en/errors.ts` AND `id/errors.ts`. Server returns `{ code, message }`, frontend `normalizeError()` picks it up.

## Migrations — the rules change once you adopt this boilerplate

This repository ships with a **single** `0000_*.sql` migration file that
represents the full baseline schema. That is deliberately squashable and
regenerate-able **only while the boilerplate is unmodified** — before you
have run `db:migrate` against any environment you care about.

The moment you start building on top of this boilerplate, migrations
become **append-only**. Full stop.

### Correct workflow — every schema change after adoption

```bash
# 1. Edit packages/db/src/schema/<file>.ts
# 2. Generate a NEW migration file. Drizzle diffs the current schema
#    against the last snapshot and writes 0001_*.sql, 0002_*.sql, ...
bun run db:generate
# 3. Review the generated SQL — drizzle-kit occasionally emits DROP + CREATE
#    when it should ALTER; catch that in review, not in production.
# 4. Commit the .sql file AND the meta/*.json snapshot together.
# 5. Apply
bun run db:migrate
```

### Never do this after adoption

- **Never delete `drizzle/0000_*.sql` and regenerate from scratch.**
  Postgres tracks applied migrations by file hash in `__drizzle_migrations`.
  Regenerating changes the hash → drizzle thinks the migration never ran
  → next `db:migrate` tries to `CREATE TABLE` on tables that already exist.
- **Never squash existing migrations.** If dev A and dev B have both
  already applied `0000` and `0001`, squashing into a new `0000` leaves
  their local DBs in a state the file no longer matches. CI is fine; the
  humans are not.
- **Never edit an already-shipped migration.** Once a migration file has
  been applied anywhere except your own laptop, treat it as immutable.
  Add a corrective `0002_fix_the_thing.sql` instead.

### Why the extra rigor

- Multi-dev: each teammate's local DB is a lightweight prod. A squash
  desynchronizes everyone silently.
- CI: the pipeline applies migrations against a clean DB each run. A
  squash usually works there → false positive, then breaks on staging.
- Production: `__drizzle_migrations` state cannot be "undone" without
  manual SQL. The recovery path is worse than doing it right the first time.

### The one exception

If you're the very first user, cloning the boilerplate on day zero,
you MAY delete `drizzle/0000_*.sql` + `drizzle/meta/*` and regenerate
after your first schema edit — collapse the boilerplate baseline + your
edit into a single fresh `0000`. Once you have deployed to any
environment beyond your laptop, this door is closed.

### CHECK constraints and enum evolution

The boilerplate ships `CHECK (col IN (...))` constraints on enum-like
text columns (`members.role`, `subscriptions.status`, `payments.status`,
`files.bucket`, `files.status`, `invitations.status`, `invitations.role`).
Adding a new enum value = a migration that drops and re-adds the CHECK:

```sql
-- Example: teach subscriptions about a 'suspended' state.
ALTER TABLE subscriptions DROP CONSTRAINT subscriptions_status_check;
ALTER TABLE subscriptions ADD CONSTRAINT subscriptions_status_check
  CHECK (status IN ('active', 'trialing', 'past_due', 'canceled', 'incomplete', 'suspended'));
```

Also update the const in `@saas/shared/constants` in the same commit —
`SUBSCRIPTION_STATUS.SUSPENDED = 'suspended'` — so app-level code can
reference it. The DB CHECK is the safety net; the const is the API.

## Non-goals

- No ORM other than Drizzle. Do not add Prisma.
- No global state library. Server state = React Query. UI ephemeral = `useState`. UI global (theme, language) = context.
- No REST client codegen. Type-safety is via OpenAPI docs for external consumers; internal is via shared `@saas/*` types.
- No CSS-in-JS. Tailwind v4 + shadcn only.
- No Docker Compose in production. It's local-dev only (`saas/saas/saas` creds).
- No `.env` file committed, ever. `.env.example` is the ONLY tracked env template.
- No hardcoded user-visible strings in components. All go through `t()`.

## Landmines — things that WILL bite you

These are wounds we already took in this codebase. Do not repeat them.

### TypeScript resolution

- **Do NOT use `.js` extensions on internal relative imports** (`from './client.js'`). Bun runtime is happy either way, but `drizzle-kit`'s CJS loader can't resolve them. Use extensionless: `from './client'`.
- **Do NOT put `composite: true` or `references` in leaf package tsconfigs.** TypeScript then expects pre-built `dist/` output for downstream imports. We run everything from source via Bun; no build step is required for `packages/*`.
- **tsconfig `extends` uses relative paths, not workspace names.** `"extends": "../config/tsconfig/node.json"`, not `"@saas/config/tsconfig/node.json"`. Workspace symlinks in `node_modules` do not resolve reliably through `extends`.

### Env loading

- **`bun run --filter <pkg> <cmd>` does not load `.env` from the workspace root.** Package scripts that need env vars must invoke `bun --env-file=../../.env run …` explicitly. See `packages/db/package.json`.
- **drizzle-kit does not auto-load `.env`.** `packages/db/drizzle.config.ts` includes a manual `readFileSync('../../.env')` parser. Do not delete it.
- **`AUTH_SECRET` cannot be the placeholder value.** The Zod schema in `packages/config/src/env.ts` rejects `change-me-to-a-random-32-char-string` at boot with a clear error. Generate real ones with `openssl rand -hex 32`.
- **`AUTH_TRUSTED_ORIGINS` is validated as URLs**: wildcards rejected, non-localhost origins must be HTTPS. Do not "quickly" set it to `*`.

### Better Auth

- **Primary key ids are 24-char nanoids, NOT 32-char Better Auth defaults.** `advanced.database.generateId: false` in `packages/auth/src/server.ts` is mandatory. Removing it breaks user creation with `FAILED_TO_CREATE_USER` silently.
- **`sessions.active_organization_id` is REQUIRED.** Without it, every organization-scoped endpoint returns "Organization ID is required". A `databaseHooks.session.create.before` seeds it from the user's first membership on sign-in.
- **`invitations` schema uses `status` + `inviter_id`, not `token` + `invited_by`.** Better Auth 1.6 expects the former exactly. Do not "clean up" the schema.
- **Better Auth 1.6's `sendInvitationEmail` payload has no `inviteLink`.** Construct the URL from `invitation.id` inside the callback. See `packages/auth/src/server.ts` for the pattern.
- **Password minimum is 8, maximum is 128.** Enforced server-side. Client `minLength` should match.
- **`requireEmailVerification: true`** in production. Only dev opts out.

### Vite + TanStack Router

- **Plugin order matters.** `tanstackRouter({ target: 'react', autoCodeSplitting: true })` MUST come before `react()` in the Vite plugin array. Wrong order causes route generation to fail silently.
- **`routeTree.gen.ts` is generated.** Do not edit. `vite build` and `vite dev` regenerate it. A stubbed placeholder is committed so typecheck passes on fresh clone before the first vite run.
- **`@vitejs/plugin-react` version must match Vite major.** Plugin v6 requires Vite 8. Do not mix.
- **One shell per subtree — never nest `<AppShell>` inside `<AppShell>`.** `apps/web/src/routes/app.tsx` already mounts `<AppShell>` around `<Outlet />` for every `/app/*` child. A child page that wraps its own return in `<AppShell>` produces a nested `SidebarProvider`, a duplicated `<AppHeader>`, and `p-4 md:p-6` padding applied twice — visually: sidebar-in-sidebar, header-in-header, content squished into a centered column. Same rule for `<SectionLayout>` at `app.account.tsx` + `app.workspace.tsx`. Dashboard child routes render `<PageHeader />` + content directly. Full explanation and the correct/wrong templates live in `apps/web/AGENTS.md` under "Layout composition". Grep to verify (closing tags — opening-tag greps also match doc comments in `__root.tsx`/`app.tsx`): `rg -n '</AppShell>' apps/web/src/routes` must return exactly one hit (`app.tsx`); `rg -n '</SectionLayout>' apps/web/src/routes` must return exactly two hits (`app.account.tsx` + `app.workspace.tsx`).

### Tailwind v4

- **`.container` in v4 does NOT center or pad.** It only sets responsive `max-width`. `apps/web/src/index.css` defines `@utility container { margin-inline: auto; padding-inline: 1rem; … }` to restore v3 behavior. Removing it makes every "container" element lengket to the left edge.
- **`tailwind.config.ts` is intentionally absent.** All config lives in `index.css` via `@theme inline`. Do not re-add a JS config file.
- **`tw-animate-css` replaces `tailwindcss-animate`.** v3 packages do not work.

### Zod v4

- **`z.record(z.unknown())` is a compile error.** Zod 4 requires two arguments: `z.record(z.string(), z.unknown())`.
- **A single Zod version is enforced.** Root `package.json` `overrides.zod` pins the whole tree. Better Auth 1.6 is also on Zod 4 — do not add packages that require Zod 3.

### Landing / marketing

- **Landing theme is DARK-scoped** via the `.landing-theme` class on `MarketingShell landing`. Dashboard tokens (`--background`, `--foreground`, etc.) are unaffected. Do NOT add a landing style outside `.landing-theme` — it will leak into the dashboard.
- **Copy is deskriptif, not marketing.** No exclamation marks, no "skip 6 months of setup", no unverifiable claims. Every version number on the landing must match `package.json`. Grep bumped versions in code before bumping them on the landing.
- **Bun 1.3 in CI + Dockerfiles + engines.** Do not lower to 1.1 without also lowering the landing.

### Errors + i18n

- **Never render raw thrown values.** Always through `normalizeError(err)` or `errorMessage(err)`. The i18n `errors.<CODE>` map handles humanization.
- **Add new error codes in TWO locale files** at the same commit. Missing key falls back to server message, then to generic `errors.fallback` — user still sees something sensible but you missed a translation.
- **QueryClient MutationCache has a global `onError`.** Mutations without their own handler still surface a toast. Opt out with `meta: { silent: true }` when the caller already shows an inline error (login form).

### Testing

- **Root-level `bun test` scans the whole workspace** — including Playwright specs, which crash on import. `package.json` `test` is scoped: `bun test packages/ && bun run --filter @saas/api test && bun run --filter @saas/worker test && bun run --filter @saas/web test`. Vitest handles web via jsdom.
- **Vitest excludes `e2e/**`** in `apps/web/vite.config.ts`. Playwright specs share the `.test.ts` extension pattern; without the exclude they'd double-run.

### Database + indexing

- **Every new query on a tenant-owned table MUST filter by `organization_id`.** Missing the predicate is a cross-tenant data leak. Grep for `organization_id` in an unfamiliar route before merging.
- **Composite indexes follow "equality first, range/sort last".** Multi-tenant queries put `organization_id` FIRST, then equality filters (`status`), then range/sort columns (`created_at DESC`). Wrong column order = index sitting on disk paying write cost but never used. Full rules in `packages/db/AGENTS.md`.
- **`ORDER BY created_at DESC` in a hot query needs `created_at.desc()` in the index.** Postgres can scan an ASC index backward, but declaring the index direction matches the query direction lets the planner drop the Sort node reliably.
- **Partial indexes for always-filtered predicates.** If EVERY hot query for a table filters on the same constant (e.g. `status = 'pending'` for invitations), the index should use `.where(sql\`... = 'pending'\`)`. Smaller, hotter in memory. Historical rows stay queryable via seq scan for admin paths.
- **Retention discipline for growing tables.** `notifications-retention` worker deletes READ notifications > 90 days (see `apps/worker/src/jobs/notifications-retention.ts`). Unread stays forever. `audit_logs` is an intentionally-unbounded compliance ledger — do not add retention there unless you have a regulatory obligation. Two tables that DO need retention added per-deployment: whatever domain-specific logs your app writes.
- **Partitioning is opt-in, not shipped.** `notifications` + `audit_logs` will hit index-depth / VACUUM costs around ~10M rows. Boilerplate ships un-partitioned because partitioning up-front has ongoing operational cost. See `packages/db/AGENTS.md` "Partitioning — when to reach for it" for the migration recipe and trigger conditions.
- **`postgres.js` `prepare: false` is required for pooler compat.** PgBouncer / Supavisor transaction-mode drops prepared statements between transactions, breaking prepared queries silently. `prepare: false` is the safe default for both direct and pooled connections.
- **Regenerate migrations from scratch is FINE while this repo is a boilerplate with no downstream deployment.** Once anyone has a live migration in production, migrations become append-only. Never squash a chain that others have already run. For this repo specifically: prefer APPEND-ONLY (`0001_*.sql`, `0002_*.sql`, ...) once the base `0000` has been shared broadly. Regeneration is still allowed but should be reserved for schema-shape rewrites — additive column adds go in a new file so existing dev DBs don't need a reset.

### Security invariants

- Every route that mutates workspace state uses `requireRole(services, session, 'admin')`. Read-only endpoints use `requireActiveOrg`.
- Every webhook: signature verified with `timingSafeEqual`, amount and currency re-checked against the pending payment row, subscription activation runs in the same transaction, terminal states (paid/refunded) are immutable — enforced by a CONDITIONAL update (`WHERE status NOT IN ('paid','refunded')` + rowcount check), not just a pre-read guard, so it holds under concurrent webhook/reconciler delivery too.
- Cookie flags are explicit in Better Auth config (`HttpOnly`, `SameSite=lax`, `Secure` if `baseURL` is https). Production asserts HTTPS at boot.
- `/api/docs` is gated behind `NODE_ENV !== 'production'`.
- Debug logs (Better Auth's `debugLogs`, stack traces in error middleware) are gated to `NODE_ENV === 'development'` — staging is silent.
- Rate limiter: `authRateLimit()` (10/min per IP) on credential-stuffing surfaces (`/api/auth/sign-in/*`, `/api/auth/sign-up/*`, `/api/auth/forget-password`) AND on email-sending/token-consuming endpoints (`send-verification-email`, `reset-password/*`, `change-email`) — those trigger Resend sends and are cost-amplification vectors. All OTHER `/api/auth/*` endpoints (get-session, organization/*) are exempt from every limit: they burst 5–15× per SPA page load and limiting them 429s ordinary navigation. General limiter on non-auth `/api/*`. Webhooks skip everything.
- Secrets-at-rest encryption is two separate mechanisms (see golden rule 9 — do NOT conflate): OAuth tokens via Better Auth's `encryptOAuthTokens` (keyed off `AUTH_SECRET`); verification tokens + integration secrets via `@saas/shared/crypto` (keyed off `COLUMN_ENCRYPTION_KEY`, required in production).

## Files that must stay in sync

| When you change...                       | Also update...                                                                            |
|------------------------------------------|-------------------------------------------------------------------------------------------|
| `packages/config/src/env.ts`             | `.env.example` and the AGENTS.md that lists env keys                                       |
| `packages/db/src/schema/*`               | Run `bun run db:generate`, commit `packages/db/drizzle/*`                                  |
| An i18n key inside namespace X           | Both `apps/web/src/i18n/locales/en/X.ts` AND `id/X.ts`                                     |
| A new i18n namespace                     | Add file + register in both `en/index.ts` and `id/index.ts`                                |
| An error code returned from the API      | `apps/web/src/i18n/locales/{en,id}/errors.ts` under `errors.<CODE>`                        |
| A payment provider signature             | `packages/payments/src/providers/<name>.test.ts`                                          |
| A payment provider's payload shape       | `redactWebhookPayload` allowlist in `packages/payments/src/providers/<name>.ts` + tests    |
| A landing claim / version number         | The corresponding line in `package.json`, Dockerfile, or CI workflow                       |
| A shadcn primitive                       | Regenerate `components.json`-driven artifacts if you use `pnpm dlx shadcn add`             |
| `packages/auth/src/server.ts` invariants | This root AGENTS.md's "Better Auth" landmine section — the schema is coupled to the plugin |
| A new file `scope` in `@saas/shared/constants.FILE_SCOPES` | A route-level role check in `apps/api/src/routes/files.ts` + any client-side upload helper docs |
| `packages/storage/src/providers/*` signed URL shape | `packages/storage/src/providers/*.test.ts` |
| `packages/db/src/schema/notifications.ts` shape | UI at `apps/web/src/routes/app.notifications.tsx` + header bell in `apps/web/src/components/app/app-header.tsx`; API in `apps/api/src/routes/notifications.ts` |
| `packages/db/src/schema/audit.ts` shape or a new `action` string | Frontend i18n `en/activity.ts` + `id/activity.ts` under `actions.<name>`; `KNOWN_ACTIONS` filter in `apps/web/src/routes/app.activity.tsx` |
| `RETENTION_DAYS` in `apps/worker/src/jobs/notifications-retention.ts` | Any customer-facing docs describing how long notifications stay in the inbox. |
| `packages/shared/src/crypto.ts` `hashApiKey` return type | `apps/api/src/middleware/session.ts` bearer lookup + `apps/api/test/harness.ts` fake key seed + `api_keys.hashed_key` column type (must stay `bytea`). |
| `packages/auth/src/server.ts` `AuthOrgAuditAction` union | Wire the new event both in the auth hook AND in the frontend i18n table under `activity.actions.<name>` |
| A new sensitive action (password change, MFA setup, account deletion) | Use `requireCookieAuth(c)` from `apps/api/src/lib/session-source.ts` to reject bearer auth |

---
> Source: [orif1n/kirim-saas](https://github.com/orif1n/kirim-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
