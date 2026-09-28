## dat-lt-command-center

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

DAT LT Outstation Command Center — a line-maintenance control application for an airline's
Italian outstation network. `README.md` is the product-level document (domain rules, API
surface, deployment); this file covers what you need to work in the code.

## Commands

npm workspaces (`server`, `web`). Run everything from the repo root.

```bash
npm run dev              # API on :8080 and Vite on :5173 (proxying /api), concurrently
npm run build            # web build, then prisma generate + tsc for the server
npm run typecheck        # both workspaces; server uses tsconfig.test.json so tests are checked too
npm test                 # server (unit + integration), then web
npm run test:unit        # server unit tests + the whole web suite — no database needed
npm run test:integration # server integration only — needs PostgreSQL
```

There is no linter. `npm run typecheck` is the static check, and it must pass before a commit.

### Running a single test

```bash
npm run test -w server -- test/integration/grounding.test.ts
npm run test -w web -- src/pages/Defects.test.tsx
npm run test -w server -- -t "releases the aircraft"    # by test name
```

### Database for integration tests

`server/vitest.config.ts` supplies `DATABASE_URL`, `JWT_SECRET` and `NODE_ENV` itself, so the
suite needs no environment variables — only a reachable database with the schema applied:

```bash
docker run -d --name datlt-test -e POSTGRES_PASSWORD=devpass \
  -e POSTGRES_DB=datlt_test -p 55433:5432 postgres:16-alpine
DATABASE_URL=postgresql://postgres:devpass@127.0.0.1:55433/datlt_test npm run db:migrate
```

That URL is the built-in default; `TEST_DATABASE_URL` overrides it. The suite truncates every
table between tests, so never aim it at a database you care about.

### Schema changes

```bash
npm run db:migrate:dev -- --name describe_the_change   # generate + apply locally
```

Commit the generated migration with the schema. Deployments run `prisma migrate deploy`, which
only plays committed migrations forward. `db:push` is for throwaway experiments only.

`@prisma/client` types come from `prisma generate`. If typecheck reports missing exports like
`Role` or `DefectCategory`, the client is stale — regenerate it rather than changing the imports.

## Architecture

### Shape

Fastify 5 API (TypeScript, ESM, Zod-validated) over PostgreSQL via Prisma, serving a React 19 /
Vite SPA from its own origin in production. One Docker image, one Railway service.

`buildApp()` in `server/src/app.ts` wires everything without binding a socket, so tests drive it
through `app.inject()`. Anything tied to the process lifetime — listening, signal handlers, the
SSE heartbeat — belongs in `index.ts`, not `buildApp`. The static-file registration must stay
last: it owns the catch-all not-found handler that falls back to `index.html`.

### The mutation convention

Every mutating route follows the same order, and new ones should:

1. Validate the body with Zod; on failure return `400` with `{ error: 'BadRequest', ... }`.
2. Read the existing row so a before/after pair can be built.
3. Write.
4. **Reconcile derived state** — grounding and release follow from the defects open against an
   aircraft, so they are recomputed after any change that could alter them, not just on close.
5. `broadcast(...)` a domain event from `events.ts`.
6. `recordAudit(request, ...)` with a rendered one-line `summary` and `changedFields(...)`.

`recordAudit` swallows its own failures on purpose (logged, never surfaced): an audit problem
must not turn a successful maintenance action into an error an engineer sees.

Routes answer every expected failure themselves with an explicit status and message. Refusals
that protect the technical log return `409` naming what is in the way. Anything that reaches the
error handler in `app.ts` is therefore a bug, and it returns a short reference instead of
`err.message` — an unhandled Prisma fault would otherwise put constraint and column names on the
wire.

### Domain invariants that are not obvious from the schema

- A `CRITICAL` defect grounds its aircraft; closing **or downgrading** the last one releases it.
  An airframe set `AOG` by hand is somebody's decision and is never overturned by an edit to an
  unrelated defect.
- The rectification deadline is **derived** from the MEL category, so re-categorising moves the
  deadline with it. A deferral replaces that window with its own expiry; bringing a deferred
  defect back to `OPEN` drops the deferral and restores the category window.
- Deferral is paperwork, not a status flip — `PATCH` rejects `status: 'DEFERRED'` and points at
  `POST /api/defects/:id/defer`.
- Defect references are allocated inside a transaction under `pg_advisory_xact_lock`. Counting
  first and inserting afterwards lets two simultaneous reports collide on the unique column and
  drops one.

### Auth

`authenticate` (`server/src/auth.ts`) verifies the token and then **re-reads the account on every
request**, so deactivation or demotion through Access Control takes effect immediately rather
than whenever a twelve-hour token expires. `sessionsValidFrom` revokes outstanding sessions;
tokens carry a millisecond `issuedAtMs` because the standard `iat` claim is too coarse to
distinguish a token from a revocation in the same second.

The SSE stream is authorised by a separate thirty-second ticket (`typ: 'stream'`), because
`EventSource` cannot set headers and a session token in a query string ends up in access logs.
`authenticate` explicitly refuses a ticket, or the narrow credential would become a broad one.

### Throttling

Requests are counted against the **account**, falling back to the network address when
unauthenticated — a station office behind one public IP would otherwise share a single budget.
Three non-obvious constraints, all learned the hard way:

- Only one limiter runs per request. `@fastify/rate-limit` sets an internal `rateLimitRan` flag,
  so a route limiter silently does nothing once the global one has run. `/login` calls
  `app.createRateLimit(...)` inside the handler to get around this.
- The global limiter must stay on the default `onRequest` hook. Moving it to `preHandler` (to
  reach a parsed body) puts it *after* `authenticate`, and unauthenticated floods stop counting.
- `createRateLimit`'s verdict is `isAllowed: false` for everything except an allow-listed caller.
  The real signal is `isExceeded`; checking `isAllowed` alone 429s every request.

Tests build the app with `rateLimiting: false`, so the `rateLimit` / `createRateLimit` decorators
are absent — guard any use with `app.hasDecorator(...)`.

### Realtime and the single-instance constraint

Mutations broadcast over SSE; `useLiveStream` in `web/src/hooks.ts` maps each event to the
TanStack Query keys it invalidates, and the affected boards refetch. Adding a `broadcast` without
a matching listener there means the change never reaches other people's screens.

Both the SSE client registry (`events.ts`) and the throttle counters live in process memory, so
**the service runs one instance**. A second replica would miss half the events and double every
limit. Moving either to Redis is the prerequisite for scaling out.

### Frontend

`useAuth().can(...roles)` is the single gate for role-dependent UI; the sidebar nav in
`Shell.tsx` filters on it. The API client (`web/src/api.ts`) treats a `401` as a dead session —
it drops the token and dispatches `datlt:unauthorized` — except on `/auth/login`, where a `401`
is the verdict on the credentials just typed.

Responses carry a CSP allowing scripts and connections from the app's own origin only, with
Google Fonts as the single exception. Anything added to the SPA that loads from a third origin
has to be allowed in `app.ts` or the browser drops it silently.

## Testing layers

Unit tests cover pure formulas (projection, MEL windows, predictive ranking, date helpers). The
web suite renders pages against a stubbed API in jsdom; `src/test/setup.ts` stubs `EventSource`,
which jsdom lacks.

Integration tests use a real PostgreSQL because the rules worth protecting — cascades, unique
constraints, the `409` refusals, reference allocation under concurrency — only exist against one.
They share a single database and run with `fileParallelism: false`.

## Conventions

Comments here explain *why*, usually naming the failure the code prevents, and the codebase reads
as prose rather than as annotated types. Match that when editing: a comment restating what the
next line does is worse than none. Commit subjects are imperative sentences naming the problem
fixed ("Stop losing defect reports to a reference collision").

---
> Source: [professor-k/dat-lt-command-center](https://github.com/professor-k/dat-lt-command-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
