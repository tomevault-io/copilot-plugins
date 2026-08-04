## fana

> Guidance for AI agents (and humans) working in this repo. This is the single source

# AGENTS.md

Guidance for AI agents (and humans) working in this repo. This is the single source
of truth — `CLAUDE.md` is a symlink to this file, so Claude Code loads it automatically
and other agents (Cursor, Codex, etc.) read `AGENTS.md` directly. **Keep it accurate —
see [Maintaining this file](#maintaining-this-file).**

## What this is

**fana** — a self-hostable disposable / temporary email service.
Random address in, mail arrives live, auto-expires. OSS, MIT.

> ⚠️ Inboxes are **public by design** — anyone who knows an address can read it.

## Stack

- **pnpm** workspaces + **Turborepo**, TypeScript strict, Node ≥ 22
- **Postgres** via **Drizzle ORM** (single dialect — MySQL support was considered and declined)
- **Redis** for realtime pub/sub + rate-limit
- Inbound **SMTP** (`smtp-server` + `mailparser`), API on **Hono** + `ws`, web on **Next.js 15** + Tailwind

## Layout

```
apps/
  smtp/   inbound SMTP → parse → store → publish realtime event
  api/    REST + WebSocket, per-IP rate-limit, admin (accounts + sessions), TTL purge job
  web/    Next.js realtime inbox UI + /admin dashboard (light default)
packages/
  core/   shared types, address generation, config + branding helpers
  db/     Drizzle schema + client (Postgres)
infra/    Caddyfile (reverse proxy, auto-HTTPS)
```

Inbound mail is authenticated at ingestion with **mailauth** (SPF/DKIM/DMARC) using
the SMTP session (IP/HELO/MAIL FROM). Results + a derived `verdict`
(`verified`/`unverified`/`suspicious`, see `packages/core/src/auth.ts`) are stored on
the message and shown as a badge in the UI. Never trust unauthenticated senders.

## Data flow

```
MX :25 → apps/smtp → simpleParser → Postgres (messages + attachments, bytea)
                                  → Redis PUBLISH event
apps/api  ← Redis SUBSCRIBE → fan out to WebSocket clients (/ws?mailbox=addr)
apps/web  ← REST (list/read) + WS (live new-mail) →
```

## Commands

```bash
pnpm install
docker compose up postgres redis -d   # datastores for local dev
cp .env.example .env                   # then set MAIL_DOMAINS
pnpm db:migrate                        # apply migrations (canonical)
pnpm dev                               # smtp + api + web in watch mode
pnpm typecheck && pnpm lint && pnpm test   # all must pass before a PR
docker compose up --build              # full stack
```

## Conventions (important)

- **Config comes from env, never hardcoded in tracked files.** Tracked files ship
  *generic* defaults (`example.com`, `localhost`); real values live in `.env` (gitignored).
  This is why `MAIL_DOMAINS`, `DB_*`, `SITE_ADDRESS` and `SITE_NAME` exist.
  Don't reintroduce a hardcoded domain or `DATABASE_URL`. Branding is the one exception
  to "generic defaults": the built-in theme in `packages/core/src/brand.ts` is the fallback
  every instance starts from, and `REPO_URL` there is a deliberate hardcoded attribution
  link — it is intentionally not themeable, so don't move it into env or the overrides.
- **DB config is discrete parts** (`DB_HOST/PORT/NAME/USER/PASSWORD/SSL`) built into a
  client in `packages/db` — not a connection URL.
- **Migrations are the source of truth** (`packages/db/drizzle/*.sql`). Edit `schema.ts`,
  run `pnpm db:generate` to create a migration, and **commit the SQL**. `pnpm db:migrate`
  applies pending migrations; the API also auto-migrates on boot (`MIGRATE_ON_START`).
  `db:push` exists only for throwaway local iteration — never rely on it for prod.
- **Table shape**: internal `id` (bigserial) PK + `created_at`/`updated_at` on every
  table. Rows exposed by the public API (messages, attachments) also carry an
  unguessable `public_id` (uuid) — the API returns/accepts `public_id`, NEVER the
  internal `id`, so nobody can enumerate rows by counting. Data is **hard-deleted**
  (ephemeral service) — no soft-delete/`deleted_at`.
- **Attachment blobs go through `@fana/storage`** (`getStorage()`), driver chosen by
  `STORAGE_DRIVER`: `db` (inline Postgres bytea, default) or `s3` (S3/R2/MinIO). An
  attachment row has exactly one of `content` (db) or `storageKey` (s3). All message
  deletion goes through `deleteMessagesWhere` so s3 objects are cleaned up too.
- **Served domains are dynamic**: built-in (`MAIL_DOMAINS`) ∪ verified community
  domains (`domains` table). SMTP + API each cache the set (`domains.ts`, refresh ~30s)
  — never query per-request. Community domains self-register via `POST /api/domains`;
  ownership is proven by the domain's MX pointing at `PUBLIC_MX_HOST` (`DOMAIN_VERIFY=off`
  auto-verifies for local dev). Reads stay public.
- **All DB access goes through `packages/db`** (`getDb()` + Drizzle). Queries currently
  live inline in routes/smtp; if that grows, extract a repository layer.
- **Addresses**: two paths.
  - *Random* (`packages/core/src/address.ts`, 3 styles, each ≥1e9 combos): minted +
    **reserved** server-side (`reservations` table), returns a token the client renews via
    `POST /mailbox/claim`. Reservation only stops the generator handing out duplicates.
  - *Explicit* (typed, or from a URL like `/alias@domain`): **public, used directly by the
    client — no reservation, no token, no "taken" error**. This is the generator.email model.
  - The web resolves URL paths to an address in `lib/resolve.ts` and mirrors the current
    address into the URL. Mailbox reads are public by design regardless of path.
  - `POST /mailbox/random` with no `domain` picks a **random served domain**, so new
    addresses spread across the instance instead of piling onto the first one.
- **Several inboxes = several browser tabs**, not an in-app switcher. `useInbox` keeps the
  address + reservation token in **sessionStorage** so each tab is independent, mirroring
  them to localStorage only as the "last used" hint for a tab opened later. Don't move that
  state back to localStorage alone — two tabs would then fight over one address.
- **`NEXT_PUBLIC_*` vars are baked at build time**, so only `NEXT_PUBLIC_API_URL` is one
  (passed as a Docker build arg — see `apps/web/Dockerfile` + compose). Don't add more:
  anything an operator may want to change belongs in runtime env or the `settings` table.
- **Branding is data, not build config.** `packages/core/src/brand.ts` owns the layering
  (overrides saved in `settings['branding']` → `SITE_NAME` env → the built-in fana theme)
  and derives every accent token from one colour in OKLCH. The API serves the effective
  brand at `GET /api/branding` and edits it under `/api/admin/branding`; the web root
  layout is `force-dynamic`, fetches it per request (`lib/site.ts`, memoized ~5s) and
  injects the accent vars as a `<style>` that overrides `globals.css`. Client components
  read it via `useBrand()`. Operator-supplied colours are re-emitted as numeric `oklch()`
  and asset URLs pass `safeAssetUrl` — never interpolate raw input into CSS or `href`.
  Client bundles must import `@fana/core/brand`, not `@fana/core` (the index pulls in
  `node:crypto` via address generation).
- **Two API surfaces, deliberately.** `/api/*` is free and keyless — the website uses it
  and its inboxes are public by design. `/v1/*` is the customer API: `requireApiKey`
  resolves the key, charges one unit of its plan's monthly quota and per-minute burst, and
  answers with `X-Quota-*`. Keys and plans are rows (`api_keys`, `plans`), never constants:
  an instance seeds one free plan and the operator adds paid ones in /admin. Usage counts
  in Redis on the hot path and flushes to `api_key_usage` every minute — bill from the
  table, never the cache. Message shape for both surfaces comes from `serialize.ts`; don't
  hand-roll a third copy. **Both are separate top-level mounts and both need a route in
  `infra/Caddyfile`** — a missing one doesn't 404, because the web app's catch-all
  resolves the unknown path to an inbox and answers 200 with HTML, so an API client sees
  a successful page instead of an error. A new top-level mount needs a `handle` block.
- **A plan belongs to the account** (`users.plan_id`), never to a key — keys inherit it,
  and quota and burst are counted per user so holding two keys can't double what a
  customer may send. The per-key counter survives only for "which key is busy" in the
  dashboard. Anything that reads a plan joins `api_keys → users → plans`; don't add a
  plan column back onto keys, or "what plan is this customer on" stops having one answer.
- **A plan's retention is real, not a label.** An inbox minted through `/v1` stores its
  `ownerUserId` on the reservation, and `apps/smtp/src/policy.ts` uses it to set `expiresAt`
  at ingestion (falling back to `MESSAGE_TTL_MINUTES`). Ownership, not `keyId`, so revoking
  a key doesn't change what an inbox it minted is entitled to — `keyId` stays only for
  "which key is busy". Changing a plan's retention only affects mail that arrives
  afterwards, and the purge job needs no changes because it only ever reads `expiresAt`.
- **Private is the default on `/v1`, and it lives on the message.** A `/v1` inbox is minted
  private unless the caller passes `private: false`; `policy.ts` then stamps
  `messages.owner_user_id` at ingestion. It is stamped on the *message*, not resolved from
  the reservation at read time, so mail doesn't turn public the moment the hold lapses.
  Every keyless surface (`/api/mailbox/*`, `/api/messages/*`) filters `owner_user_id IS
  NULL` and the public WebSocket drops events flagged `private` — it has no auth, so it
  can't do anything else. `/v1` reads see public rows plus their own account's, which makes
  somebody else's private message a 404 rather than a 403. A new read path over `messages`
  picks one of those two filters — the one exception is `/api/admin/messages`, which lists
  every row because the operator owns the database anyway; it marks them `private` and
  stays metadata-only, so don't grow a body-reading admin route.
- **`concurrentInboxes` is enforced where inboxes are minted** (`POST /v1/inboxes`), counted
  over live reservations for the *account*. An API-minted address is held for the plan's
  retention (floor 30 min) rather than `RESERVATION_TTL_MINUTES` — that number belongs to
  the browser client, which renews; an API caller calls `/v1/inboxes/:address/release`
  instead, which is also what frees a slot early. Because a limit you can hit needs
  somewhere to act on it, the same list and release are reachable with a session at
  `/api/account/inboxes` and shown in the customer's Inboxes section. Both surfaces answer
  from `inboxes.ts`, and both message filters live in `visibility.ts` — a read path over
  `messages` imports `publicOnly` or `visibleTo` rather than writing the condition again.
- **The sender is blocked until `handleMessage` returns**, so that path only does what
  durability needs. Parse, SPF/DKIM/DMARC and the per-mailbox policy lookup run in one
  `Promise.all` (none needs another's output); attachment blobs upload while the message
  row is being written, since only the attachment *rows* need its id; the hourly stats
  counter is fired and not awaited. What must stay awaited is anything a 250 promises:
  the message row and its attachment rows. Don't ACK before those land, and don't add a
  new `await` between starting the uploads and the `Promise.all` that collects them — the
  gap would leave those rejections unhandled. SPF/DKIM/DMARC stays on the path on purpose:
  moving it off means publishing a message whose verdict isn't known yet, and a badge that
  changes after the fact is worse than the ~15ms.
- **A decision worth testing gets its own pure module.** `@fana/core` is pure by
  construction, and the same rule now applies inside the apps: what a message *becomes*
  (`smtp/message.ts`), whether a sender may proceed (`smtp/ratelimit.ts`), who a request
  is charged to (`api/ratelimit.ts`) and how a long-poll settles (`api/events.ts`) are
  separated from the database, Redis and Hono around them. Env-dependent helpers take
  `env: NodeJS.ProcessEnv = process.env` rather than reading a module-level constant —
  a constant captured at import can only ever be exercised at one value. Redis-touching
  helpers accept a narrow interface (`CounterRedis`, `ScreenRedis`) so a test hands in a
  fake instead of needing a server. Don't inline one of these back into its caller.
- **Codes and links are extracted on read, not at ingestion** (`@fana/core/extract`,
  pure and heavily unit-tested). Nothing is stored: the heuristics will keep changing,
  and a column would freeze whatever they said the day the mail arrived. `serializeMessage`
  (one message) includes `extracted`; `serializeMessages` (a listing) does not — that
  split is the whole cost control, so don't "helpfully" add it to lists. Ranking is by
  *distance* to a word naming the code, never a boolean "is a keyword nearby": in
  "Order 88991122 … your code is 445566" both are near one. The module must stay free of
  `node:crypto` and the DOM so the browser can import `@fana/core/extract` directly.
- **A webhook endpoint belongs to the account, never to a key.** A message records
  `owner_user_id` and nothing else, so the account is the only thing the dispatcher can
  resolve — and an endpoint tied to a key would stop delivering the moment that key was
  rotated, for inboxes still alive. Enqueue runs in the **API**, not SMTP: the sender is
  blocked until `handleMessage` returns and somebody else's HTTP server is not that path.
  The payload is snapshotted into `webhook_deliveries` at enqueue because the message can
  be purged before a retry succeeds. Every node sees the same event and every one tries to
  enqueue, so the unique index on `(webhook_id, message_id)` is what makes the rest no-ops
  rather than duplicate POSTs, and the deliverer claims rows with `FOR UPDATE SKIP LOCKED`.
  A customer-supplied URL makes this server fetch whatever they name, so
  `checkWebhookUrl` rejects private literals at registration **and** `deliver.ts` resolves
  the hostname again before connecting and never follows a redirect — the first check
  alone is bypassed by a name that resolves privately or is repointed later.
- **One Redis subscriber per process** (`events.ts`). The WebSocket fan-out and every
  long-polling `/v1/.../wait` register listeners on that one bus — a subscriber
  connection per waiter would not survive any real traffic. An event is only a *wake-up*:
  `waitFor` re-runs a DB probe rather than reading the payload, so a waiter never has to
  trust something that crossed a pub/sub channel to decide what it may see. It registers
  the listener *before* the first probe, or a message landing between the two would fall
  into the gap and the call would time out with the mail sitting in the inbox.
- **One `users` table, two roles.** Operators and customers share it because an
  operator wants a key too and both need the same sessions; `role` (`admin` | `customer`)
  is the only difference, and `password_hash` is nullable because a customer signs in
  through a provider. OAuth logins are rows in `identities` (provider + provider_user_id),
  matched by provider id and **never** by email — an account can change its email, and
  trusting that would let someone claim an existing account. Providers are data in
  `auth/providers.ts`: adding Google or Discord is one object plus `<ID>_CLIENT_ID` /
  `<ID>_CLIENT_SECRET`, not a new route. The callback returns the session in the URL
  *fragment*, which never reaches a server log or a Referer header.
- **A sub-app's `use("*")` is middleware for the whole mount prefix, not for its own
  routes.** Mounted at `/api`, it runs for every `/api/*` path registered *after* it —
  which has now caused two production bugs: the account guard answering "sign in to
  manage your account" for `/api/admin/*`, and the 20/min sign-in budget being spent by
  the dashboard, locking an operator out of their own instance after a few clicks. Give
  a sub-app either its own prefix (`app.route("/api/account", …)`) or named paths
  (`use("/admin/login", …)`), never `use("*")` on a shared one. The symptom is always a
  route answering with something that belongs to a different feature.
- **Three gates, three scopes.** `requireSession` = signed in (or the instance token);
  `requireAdminRole` on top of it for `/api/admin/*`; `/api/account/*` is session-only and
  every query is scoped to `session.userId`, so guessing another user's key id gets a 404
  rather than their key. Don't reach for `/admin` routes to implement a customer feature.
- **Admin auth lives in the DB, not env.** `admins` (bcrypt hashes) + `api_tokens`
  (SHA-256 of a high-entropy token, plaintext shown once). `apps/api/src/auth/` owns it:
  `passwords.ts`, `sessions.ts` (opaque token → Redis, sliding TTL, revocable per admin),
  `tokens.ts`, `lockout.ts` (per-IP **and** per-username failure counters), `seed.ts`
  (first boot creates both and prints them once). Every `/api/admin/*` route is mounted
  behind `requireAdmin` + its own rate-limit bucket in `index.ts` — handlers must not
  re-check auth, and new admin routes go under that mount, never beside it. Login stays
  outside the guard with its own bucket. Never log or return a password or a token hash;
  a missing username still runs a bcrypt compare (`DUMMY_HASH`) so timing says nothing.
- **A list that can grow is paged at the endpoint** (`pagination.ts`: `pageParams` +
  `paged`). Offset-based on purpose — these back dashboard tables where "page 7 of 12"
  and a total are the point, unlike a feed nobody counts. `perPage` comes from a query
  string, so it is clamped: `/admin/users` used to return *every* customer row. The
  response keeps its named array (`users`, `messages`, `community`) and adds
  `page/perPage/total/pages`, so a client that ignored paging still works. On the web,
  `DataTable` renders a list whose columns are data and `Pager` is separate for the lists
  that aren't tables — the customer list expands rows to show each account's keys.
  Sorting is *not* offered client-side: it would sort the ten rows on screen out of four
  hundred, which is worse than not having it. Page belongs in the resource key
  (`` `${refreshKey}:${page}` ``) so turning a page refetches, and changing a filter
  resets to page 1.
- **UI primitives are Radix + our tokens** (`components/ui/`: Button, Badge, Input,
  PasswordInput, Textarea, Field, Dialog, DropdownMenu, Tooltip, Switch, Select,
  Skeleton, CopyButton, Toaster). Use them instead of hand-rolling a dropdown/modal/field
  — a hand-rolled one loses focus handling and the exit animation. Don't pull in shadcn's
  theme variables: colours come from the OKLCH tokens in `globals.css`, radii from
  `--radius-*`, and motion from the `anim-*` classes there (paired open/closed keyframes,
  all disabled under `prefers-reduced-motion`). Feedback goes through `toast`, not an
  inline status paragraph.
- **One dashboard at `/dashboard`, sections filtered by role.** `DashboardPage` holds a
  single registry (`adminOnly` marks operator sections) and builds both the nav and the
  route from it; `DashboardShell` owns the auth gate, sidebar, header and refresh, and
  publishes the role through `RoleContext`. A new screen is a registry entry, not a
  layout. The role check there only avoids rendering a page that would answer 403 — the
  API is the actual gate, so never rely on it for anything but the UI.
- **The API reference is a page of the app, not a separate site** (`/docs`, rendered
  from `lib/docs.ts`). It has to state *this* deployment's base URL, *this* instance's
  served domains and *this* operator's plan limits — a static docs site would be built
  against one deployment and be wrong for every self-hoster, so it reads all three per
  request (`GET /api/domains`, `GET /api/plans`, both public and keyless). The endpoint
  list is **data, not markup**: adding an endpoint is adding an entry, and moving to an
  OpenAPI-generated reference later only changes where that array comes from. It is also
  the one page meant to be indexed. Keep it and the README in step — the README is the
  short version for people reading the repo, the page is the reference.
- **Two pages are indexed, and only two**: the root and `/docs`. Everything else this
  app serves is an ephemeral inbox, a session-gated dashboard or the unadvertised
  operator path. The catch-all decides per request in `generateMetadata` — it used to be
  one blanket `index: false` covering the whole route, which kept the dashboard out of
  search and took the home page with it. The `og:image` is drawn per instance
  (`lib/ogImage.tsx`) for the same reason the docs are: a self-hoster's link preview must
  carry their name and accent. Convert the accent with `oklchToHex` first — the renderer
  behind `next/og` understands sRGB only and drops an `oklch()` string silently — and
  don't set `fontWeight`, since it ships one face at one weight.
- **One page for the reference, a page each for guides.** A reference is scanned, not
  read: one page means Ctrl+F finds everything and there is no "which page was that on".
  Endpoint count doesn't change that — don't split `/docs` per endpoint. Guides are read
  start to finish and become `/docs/<slug>` when they exist. Two rules bound what grows
  here: anything only true of the *hosted* deployment (prices, terms, blog) must not ship
  in a self-hoster's instance, and per-endpoint depth (request/response schemas, per-field
  types) waits for `@hono/zod-openapi` — hand-written schemas for every route are a drift
  machine, and depth and generation are the same decision.
- **The admin path is an entrypoint, not an area.** It serves `AdminEntry` (the operator
  password form) and redirects to `/dashboard` once signed in. Don't move operator pages
  back under it: `POST /api/admin/login` is at a fixed URL anyway, so hiding pages buys
  nothing, while hiding the form keeps scanners off it. Customers sign in at `/login`.
- **The dashboard's URL is data**, stored in `settings['admin_path']` and edited in
  Access → Dashboard URL (`ADMIN_PATH` env only seeds a fresh instance). It therefore
  can't be a folder on disk: the inbox catch-all (`app/[[...slug]]/page.tsx`) asks the API
  `GET /api/admin-path/check?path=` whether the first segment is the dashboard, then
  renders `AdminPage`, which lazy-loads one section per URL (`/{path}/inbox`, …) inside
  `AdminShell`. Never add an endpoint that *returns* the path unauthenticated, and don't
  hardcode `/admin` in links — use the `basePath` the shell is given. Sections read the
  header's refresh counter from `useRefreshKey()` and fetch through `useAdminResource`
  (load / busy / error / reload), never their own `useEffect`.
- **Git**: feature branch → PR → **squash merge**. Conventional Commits. No AI co-author trailer.

## Known gaps / gotchas

- ✅ **HTML email is sanitized** at ingestion (`@fana/core/sanitize`, sanitize-html) and
  rendered in a sandboxed iframe (`MessageModal`). Keep both layers if you touch email rendering.
- Port **25** must be open on the host; many providers block it by default.
- Inbound mail is flood-guarded per client IP (`apps/smtp/src/ratelimit.ts`, Redis
  fixed-window, `SMTP_RATE_LIMIT_*`); over-limit senders get a retryable 451.
- Attachments are stored in Postgres `bytea` — fine for small scale, move to object storage later.
- See [ROADMAP.md](ROADMAP.md) for the planned order of work.

## Maintaining this file

Treat doc drift as a bug. **In the same PR** that changes any of the following, update this file:

- Add / rename / remove an app or package
- Change a dev, build, test, or DB command
- Add / rename / remove an env var (also update `.env.example`)
- Change the architecture or data flow
- Introduce or change a convention, or close a "known gap"

If a change makes a statement here false, fix the statement — don't leave it stale.

---
> Source: [JastinXyz/fana](https://github.com/JastinXyz/fana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
