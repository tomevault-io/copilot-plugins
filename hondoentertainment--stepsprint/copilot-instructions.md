## stepsprint

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

StepSprint is a month-long step-challenge platform with teams, daily step submissions, weekly/monthly leaderboards, and an admin console. Monorepo with a React client, Express API, and Prisma database (SQLite in dev, Postgres planned for prod).

Note: `package.json` still names the root workspace `schaffer-shufflers` — legacy demo name. User-facing product name is **StepSprint**.

## Workspace layout

```
client/             Vite + React 19 + TypeScript frontend (React Router 7)
  public/icons/     PWA icons (192, 512)
  e2e/              Playwright specs (desktop + mobile)
server/             Express 5 + TypeScript API (JWT auth, Zod validation)
  prisma/           Canonical Prisma schema (SQLite active;
                    Postgres variant at schema.postgresql.prisma), migrations
  src/              App code, routes, middleware, logger, sentry, openapi
  src/seed.ts       Seed entry point (wired via server/prisma.config.ts)
api/                Vercel Function entry (api/[...all].js) — wraps the
                    compiled Express app at /api/*
docs/               LAUNCH.md, DEPLOYMENT.md, PRODUCTION.md, design notes
.github/workflows/  CI (server + client tests + lint + build + smoke + E2E)
scripts/            vercel-build.mjs (Vercel build orchestrator),
                    check-api-health.mjs, switch-to-postgres-schema.mjs
vercel.json         Single Vercel project (SPA + Function + Cron)
```

## Commands

Run from repo root unless stated:

```bash
npm test                # client + server unit/integration (Vitest)
npm run test:coverage   # both with coverage
npm run test:e2e        # Playwright (desktop + Pixel 5 + iPhone 13)
npm run lint            # client eslint
npm run build           # client + server build
npm run check:api       # npm run check:api -- https://your-host [ --strict ]; launch gate validates release + transactionalEmail + CORS when --strict
npm run postgres:parity # swap Prisma to Postgres files for local parity (see docs/DEPLOYMENT.md)
```

Per-workspace:

```bash
cd server && npm run dev          # API on :3001 (tsx watch)
cd server && npm run db:migrate   # prisma migrate dev
cd server && npm run db:seed      # admin + sample users + demo challenge
cd client && npm run dev          # Vite on :5173
cd client && npm run lint         # eslint
cd client && npm run build        # tsc -b && vite build (emits PWA SW)
cd client && npm run test:e2e:ui  # Playwright UI mode
```

Required env (see `.env.example`):
- `JWT_SECRET` — min 16 characters in development; **min 32 in production** (enforced at startup; Vitest uses a longer CI test secret)
- `DATABASE_URL` — defaults to `file:./dev.db` (SQLite)
- `APP_ORIGIN` — CORS origin for the client
- Optional: `SENTRY_DSN`, `VITE_SENTRY_DSN`, `VITE_POSTHOG_KEY`, `LOG_LEVEL`, SMTP vars

Seed users after `db:seed`:
- Admin: `admin@stepsprint.local`
- Participant (used by E2E): `user1@stepsprint.local`
- Password (all seeded users): `password123`

## Architecture

### Server (`server/src`)
- `app.ts` / `index.ts` — Express wiring: helmet → pino-http (request IDs) → rate limiters → CORS → JSON → routes → Sentry error handler → 404
- `routes/` — modular routers: `auth`, `admin`, `challenges`, `submissions`, `leaderboards`, `summary`, `analytics`, `invites`, `notifications`, `integrations`, `openapi`. Tests colocate (`*.test.ts`, Supertest)
- `middleware/auth.ts` — JWT verification + role gates (`ADMIN` / `PARTICIPANT`)
- `middleware/rateLimit.ts` — four tiers: `authLimiter`, `apiLimiter`, `generalLimiter`, `passwordResetLimiter` (prod only)
- `logger.ts` — pino (pretty in dev, JSON in prod), level from `LOG_LEVEL`
- `sentry.ts` — `initSentry()`; no-op without `SENTRY_DSN`
- `openapi.ts` + `routes/openapi.ts` — OpenAPI 3 spec at `/api/openapi.json`, Swagger UI at `/api/docs`
- `config.ts` — Zod-validated env loader (`JWT_SECRET`, `DATABASE_URL`, `APP_ORIGIN`, email `RESEND_API_KEY`/`SMTP_*`, optional `ALLOW_PRODUCTION_WITHOUT_EMAIL` for non-public prod)
- `services/` — email (Nodemailer; no real SMTP provider configured)
- `utils/` — Luxon date helpers, bcrypt, reset tokens
- `prisma.ts` — Prisma client via `@prisma/adapter-better-sqlite3`, reads `DATABASE_URL`
- Input validation: Zod schemas on every mutating route
- Errors: custom `ApiError` class; central handler converts to JSON

### Client (`client/src`)
- `main.tsx` — `initSentry()` → React root wrapped in `ErrorBoundary` → `<App>`; registers PWA SW in prod
- `App.tsx` — router + providers; admin/leaderboard/standings routes are `React.lazy` code-split with `<Suspense>` fallback
- `api.ts` — fetch wrapper, `ApiError`, auth token handling
- `components/` — page-level (`Home`, `Admin`, `Login`, `WeeklyLeaderboard`, `Submit`, `TeamStandings`, `InvitePage`, `ForgotPassword`, `ResetPassword`) and UI (`Layout`, `ConfirmDialog`, `WeekPicker`, `ErrorBoundary`)
- `contexts/` — `WeekContext` for the selected calendar week
- `hooks/` — `useAuth`, `useChallenges`, etc.
- `sentry.ts` — `initSentry()` + `captureException` helper; `ErrorBoundary.componentDidCatch` forwards here
- `analytics.ts` — `track()` / `identify()` abstraction; lazy-imports `posthog-js` only if `VITE_POSTHOG_KEY` set; wired in `Login` (identify on success), `Submit` (track on submission), `Home` (track on challenge view)
- `i18n/` — react-i18next setup; `en.json` and `es.json` hold user-visible copy; `LanguageSwitcher` exposes a select control (persists `stepsprint-locale`)
- Styling: CSS-in-JS + `App.css` / `index.css`; no UI framework
- PWA: `vite-plugin-pwa` with `registerType: 'autoUpdate'`; manifest inlined in vite.config; icons in `public/icons/`

### Data model (Prisma)
`User` (role), `Challenge` (month-scoped, timezone, `inviteCode`, `inviteCodeExpiresAt`), `Team`, `TeamMember`, `StepSubmission` (unique on `userId+challengeId+date`), `PasswordResetToken`, `AuditLog`, `NotificationPreference`.

Team assignment supports random and snake-draft at challenge creation.

### CI / Deploy
- `.github/workflows/ci.yml` — server tests (SQLite + Postgres parity), client lint/test/build, smoke health check, Playwright E2E (desktop Chrome) on PR and push to `master`.
- **Single Vercel deploy** (SPA + API Function + hourly Vercel Cron). The build runs `scripts/vercel-build.mjs`: swap to Postgres schema → `prisma generate` + `migrate deploy` → `tsc` (server) → `vite build` (client). Postgres comes from the **Vercel Marketplace Neon** integration; the build script aliases `POSTGRES_PRISMA_URL` / `POSTGRES_URL_NON_POOLING` to `DATABASE_URL` / `DIRECT_URL`.

## Conventions

- **TypeScript strict** in both workspaces — don't loosen it.
- **Zod first**: validate every request body/params; derive types from schemas where practical; register the Zod schema with OpenAPI where public.
- **Timezone-aware dates**: use Luxon and the helpers in `server/src/utils` — never `new Date()` for challenge-day arithmetic.
- **Tests colocate**: server tests sit beside routes as `*.test.ts`; client unit tests as `*.test.tsx`; Playwright specs live under `client/tests/`.
- **Audit admin writes**: admin mutations should append to `AuditLog`.
- **Prisma transactions** for multi-row writes (team allocation, submission edits).
- **Error handling**: throw `ApiError` on the server; catch `ApiError` in the client `api.ts` layer and surface typed messages; the client `ErrorBoundary` forwards render-time errors to Sentry.
- **Logging**: use the exported `logger` (pino) — no `console.log` in production code paths.
- **Analytics**: use `track()` / `identify()` from `client/src/analytics.ts`; keep event names stable — use `ANALYTICS_EVENTS` for values sent from components (see `analytics.ts`). In production builds, PostHog initializes only after the user accepts optional analytics in the cookie banner (see `CookieConsentBanner.tsx`).
- **No emojis** in code or UI unless the user asks.

## Known gaps / gotchas

- **Sentry and PostHog**: Server Sentry is silent until `SENTRY_DSN` is set. Client Sentry needs `VITE_SENTRY_DSN`. Optional **browser source maps**: set `SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, and `SENTRY_PROJECT` at Vite build time so `@sentry/vite-plugin` uploads hidden maps (see `client/vite.config.ts`). PostHog loads only when `VITE_POSTHOG_KEY` is set **and** (in production) the user accepts analytics in the banner; in development, analytics runs unless the user chose "Essential only".
- **No real SMTP provider**: Nodemailer wired, password reset emails no-op without SMTP env.
- **i18n**: Most screens use `useTranslation` with `en.json`; Spanish lives in `es.json` — language switcher in `LanguageSwitcher` (persists `stepsprint-locale` in `localStorage`).
- **Health-sync integrations**: Apple Watch via Shortcuts + bearer token to `POST /api/integrations/apple-health`; Fitbit, Google Fit, and Garmin via OAuth when server env credentials are set (`routes/oauth.ts`, `routes/integrations.ts`). In-browser HealthKit/Health Connect pairing is not a PWA goal (see `docs/PRD.md` stretch decision).
- **Push notifications**: Daily reminders can use Web Push when VAPID keys are configured; email when SMTP/Resend is configured. The hourly sweep runs via Vercel Cron (`vercel.json` → `GET /api/cron/reminder-sweep`); the in-process scheduler in `services/scheduler.ts` is a no-op when `VERCEL=1`. Bearer secret is `CRON_SECRET` (legacy `REMINDER_CRON_SECRET` still accepted).
- **CSP**: API responses use helmet with pinned CSP (strict for API routes; relaxed only for `/api/docs` and `/api/openapi.json` when OpenAPI docs are enabled).
- **CSRF**: Production uses double-submit cookie validation on `/api/*` (except Bearer-auth and specific auth endpoints); the SPA fetches `/api/csrf-token` and sends `x-csrf-token` (`server/src/app.ts`, `client/src/api.ts`).
- **OpenAPI / Swagger**: Disabled by default when `NODE_ENV=production`. Set `OPENAPI_DOCS_ENABLED=true` on the server to expose `/api/docs` and `/api/openapi.json`.
- **Deploy**: See `docs/LAUNCH.md` (ordered runbook), `docs/DEPLOYMENT.md` (deep dive), and `docs/PRODUCTION.md` (security/compliance review). The Vercel build runs `prisma migrate deploy` against the Marketplace Neon Postgres direct URL before the Function ships.
- **Dependency vulnerabilities**: Root `npm audit` clean after `npm audit fix`. In **client**, `vite-plugin-pwa` / `workbox-build` / `serialize-javascript` still report high until a non-breaking upgrade path exists (avoid `npm audit fix --force` without testing the PWA build). In **server**, moderate advisories in `@prisma/dev` → `@hono/node-server`; fixing cleanly may require a Prisma major alignment — verify before forcing.
- **Bundle size**: `Admin`, `WeeklyLeaderboard`, `TeamStandings` are code-split; `Home`, `Login`, `Submit` stay eager (critical path). Initial bundle is ~97 KB gzipped — room for more splitting, image optimization, and response caching.

## When making changes

- Run the relevant workspace's tests (`npm run test:run`) before declaring done; run Playwright when touching auth, submissions, leaderboards, or admin flows.
- Lint the client (`cd client && npm run lint`) after UI changes.
- If you change the Prisma schema, generate a migration (`db:migrate`) and update `seed.ts` if the shape of seed data changes. Keep `server/prisma/schema.prisma` and `server/prisma/schema.postgresql.prisma` in sync until one is removed.
- If you add public routes, register their Zod schemas with the OpenAPI registry in `server/src/openapi.ts`.
- If you log, use the pino `logger` — do not reintroduce `console.log` in production code paths.
- Don't rename the root package from `schaffer-shufflers` as part of unrelated work — it appears in lockfiles and would create noisy diffs; do it as its own change.

---
> Source: [hondoentertainment/StepSprint](https://github.com/hondoentertainment/StepSprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
