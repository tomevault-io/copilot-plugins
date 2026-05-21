## img-hosting

> Project-level instructions for Claude Code working in this repo.

# CLAUDE.md

Project-level instructions for Claude Code working in this repo.

## What this is

A private image host on **Cloudflare Workers**, exposing an Imgur-shaped REST
API. Single bearer token, single owner (`owner = 'me'`). Optimised for
"my notes / my screenshots / my bot" usage, not a multi-tenant SaaS.

## Stack and bindings

| Concern | Choice | Binding |
|---|---|---|
| Runtime | Cloudflare Workers (Hono router) | — |
| Image bytes | R2 | `IMG_BUCKET` |
| Metadata | D1 (SQLite) | `IMG_DB` |
| Rate limiting | Cloudflare `ratelimit` (unsafe binding) | `RL` |
| Image resize | Cloudflare image transformations via `cf.image` | — |
| Auth (CLI / agents) | Single bearer token in env | `API_KEY` |
| Auth (Web UI) | Cloudflare Access JWT (Zero Trust) | `ACCESS_TEAM` + `ACCESS_AUD` |
| Tests | Vitest + `@cloudflare/vitest-pool-workers` | — |

## Three-surface design

The same Worker answers on three hostnames with three auth profiles:

```
upload-image.example.com   ← Access JWT  (browsers; UI + /3/* + /whoami)
*.workers.dev /3/*         ← bearer      (CLI / agents)
*.workers.dev /i/:id.ext   ← none        (public image bytes)
```

`PUBLIC_BASE_URL` is set to the workers.dev URL on purpose: the `link`
field in every upload response points at the **public** hostname so
shared image URLs don't bounce viewers through Access. Don't change
`PUBLIC_BASE_URL` to the custom domain unless you also add an Access
**Bypass** policy scoped to `Path: /i/*`.

## File map

```
src/
  index.ts        Hono root router, mounts sub-apps, error handler,
                  serves /whoami for the UI identity probe.
  env.ts          `Env` interface (bindings + ACCESS_TEAM/ACCESS_AUD).
  auth.ts         `requireAuth` (alias `requireBearer` kept for compat):
                  tries Access JWT first, falls back to API_KEY bearer.
  access.ts       Verifies Cf-Access-Jwt-Assertion against the team's
                  JWKS. Caches keys for an hour. Only runs when both
                  ACCESS_TEAM and ACCESS_AUD are set.
  ui.ts           Single-page minimalism UI served from `/` for text/html
                  clients; /3/* JSON listing kept for everything else.
  ids.ts          base62 image id (7 chars) + deletehash (15) + sha256.
  sniff.ts        Magic-byte mime detection + dimension parsing
                  (PNG, JPEG, GIF, WebP). No external dep.
  response.ts     Imgur `{ data, success, status }` envelope helpers.
  ratelimit.ts    Per-key/per-IP wrapper around the RL binding.
                  Falls open if RL is unbound (test runtime).
  resize.ts       `?w=&h=&fit=&q=` -> `fetch(.., { cf: { image: ... } })`.
  images.ts       POST/GET/DELETE/UPDATE `/3/image[/...]`.
  account.ts      GET `/3/account/me/{images,images/count,image/:id}`.
  serve.ts        Public `/i/:filename` + internal `/raw/:filename`
                  (the latter is the source URL cf.image fetches from).

migrations/0001_init.sql   Wrangler-managed D1 migration.
schema.sql                 Canonical schema (mirrors the migration).
test/
  helpers.ts      Inlined schema (Workers test runtime has no fs),
                  tinyPng fixture, AUTH header, resetState().
  *.test.ts       upload / get / delete / account suites.
  env.d.ts        Extends `cloudflare:test` ProvidedEnv with `Env`.
wrangler.toml.example   Committed template (no real IDs).
wrangler.toml           Local-only, gitignored, contains real database_id.
.env / .dev.vars        Local-only, gitignored. Wrangler dev uses .dev.vars.
.env.example / .dev.vars.example   Committed.

img-hosting/            Claude Code skill + CLI.
  SKILL.md              Skill manifest.
  bin/img-hosting       Bash CLI wrapping the API (chmod +x).
  .env / .env.example   Skill-local config; .env gitignored.
```

The CLI is the *only* sanctioned way to call the API from shell / agents.
Don't add a Node-based CLI in parallel - keep one source of truth.

## Commands

```bash
pnpm install
pnpm dev           # wrangler dev on :8787
pnpm test          # full vitest run (isolatedStorage:false, singleWorker)
pnpm test:watch
pnpm typecheck     # tsc --noEmit, strict
pnpm db:local      # apply schema.sql to local D1 (one-time)
pnpm db:remote     # apply schema.sql to remote D1
pnpm deploy        # wrangler deploy
```

## OSS / security invariants

These are non-negotiable when editing this repo:

1. **No real Cloudflare IDs in committed files.** `database_id`,
   `account_id`, real bucket names tied to a person, etc. all live in
   `wrangler.toml` (gitignored). The committed template is
   `wrangler.toml.example`.
2. **No real secrets in committed files.** `API_KEY` lives in `.dev.vars`
   (gitignored) locally and `wrangler secret put API_KEY` in prod.
3. Tests must not require remote Cloudflare access. They run in miniflare
   with `isolatedStorage: false` and `singleWorker: true`.
4. Bearer comparison uses constant-time equality (`auth.ts:timingSafeEqual`).
   Don't replace it with `===`.

## API contract (Imgur-shaped envelope)

All API responses use `{ data, success, status }`. The deletehash field is
returned only on `POST /3/image` and `GET /3/account/me/*` — never on the
public `GET /3/image/:id`.

| Verb | Path | Auth |
|---|---|---|
| `POST` | `/3/image` | Bearer |
| `GET` | `/3/image/:id` | public |
| `POST` | `/3/image/:deletehash` | Bearer (update title/desc) |
| `DELETE` | `/3/image/:deletehash` | Bearer |
| `GET` | `/3/account/me/images?page=&perPage=` | Bearer |
| `GET` | `/3/account/me/images/count` | Bearer |
| `GET` | `/3/account/me/image/:id` | Bearer |
| `GET` | `/i/:id.:ext[?w=&h=&fit=&q=]` | public |
| `GET` | `/raw/:id.:ext` | public (used by cf.image) |
| `GET` | `/healthz` | public |

Upload accepts four body shapes (mirrors Imgur):
- `multipart/form-data` with `image` field (File or string)
- `application/x-www-form-urlencoded` with `image=<bytes-or-b64>` + optional `type=base64`
- `application/json` with `{ image, type, name?, title?, description? }`
- raw bytes (any `Content-Type` not above)

`MAX_BYTES = 20 MiB`. Supported types: PNG, JPEG, GIF, WebP. Anything
sniff-rejected returns 415.

## Data model

```sql
CREATE TABLE images (
  id TEXT PRIMARY KEY,         -- 7-char base62 (e.g. Dwql3jX)
  deletehash TEXT UNIQUE NOT NULL,  -- 15-char base62 secret
  owner TEXT NOT NULL,         -- always 'me' in v1
  filename, title, description TEXT,
  mime TEXT NOT NULL, ext TEXT NOT NULL,
  size INTEGER NOT NULL,
  width INTEGER, height INTEGER,
  sha256 TEXT NOT NULL,
  created_at INTEGER NOT NULL,
  deleted_at INTEGER           -- soft-delete sentinel
);
```

Soft delete only. R2 object is hard-deleted at delete-time but the row
stays so historical ids 404 cleanly without colliding with new uploads.

## Conventions / preferences

- **Hono** for routing. Sub-apps mounted under `/` in `index.ts`.
- TS strict mode. No `any`. `unknown` + narrow at boundaries.
- One file per concern. Don't add a generic `utils.ts`.
- Errors flow up to `app.onError`; handlers return 4xx with `fail()`.
- Magic numbers explained in code only when non-obvious (see `sniff.ts`).
- No comments narrating *what* — comment only when the *why* is hidden.

## Gotchas learned the hard way

- **Workers test runtime has no Node `fs`.** Don't `readFileSync` in tests;
  inline test fixtures (see `test/helpers.ts`).
- **`vitest-pool-workers` defaults to `isolatedStorage: true`,** which means
  each test gets a fresh D1 — `beforeAll(applySchema)` doesn't persist.
  We set `isolatedStorage: false` and reset between tests.
- **`crypto.subtle.digest` rejects `SharedArrayBuffer` views.** Copy into
  a fresh `Uint8Array` before hashing (see `ids.ts:sha256Hex`).
- **`cf.image` transformations only run on a zone**, not `*.workers.dev`.
  On workers.dev the resize call returns the original bytes (graceful
  no-op), so the same code path works in dev without flagging.
- **`wrangler secret put` is non-interactive when stdin is piped** — uses
  the piped value as the secret. Don't accidentally pipe an empty string.
- **R2 needs explicit account activation** with both ToS *and* overage
  billing authorization checkboxes — the second one is easy to miss.

## When extending

- **New endpoint** -> add a route in `src/<area>.ts`, mount in `index.ts`,
  add a Vitest spec. Don't fork the response envelope.
- **New body shape on upload** -> extend `readBody()` in `images.ts`,
  keep the return type `{ bytes, filename, title, description } | { error }`.
- **New image format** -> add a `sniff<Foo>()` in `sniff.ts` and wire it
  into `sniffImage()`. Update `MAX_BYTES` only with care.
- **Multi-user mode** -> change `owner = 'me'` constant + add an
  owner-resolution step in `auth.ts`. Index `idx_images_owner_created`
  already supports it.

## Don't

- Don't introduce a build step beyond `wrangler` (no Webpack/Vite for src).
- Don't add a frontend in this repo. If a UI is needed, it lives in a
  separate repo that consumes this API.
- Don't replace the `ratelimit` binding with a custom Durable Object
  unless someone actually needs per-route policies.
- Don't store anything user-identifying beyond what's in the schema.

---
> Source: [htlin222/img-hosting](https://github.com/htlin222/img-hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
