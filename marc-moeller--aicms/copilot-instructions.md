## aicms

> This is the one doc to read before changing code in this repo. It distills the

# AGENTS.md — canonical engineering reference

This is the one doc to read before changing code in this repo. It distills the
architecture, the data model, and the sharp edges that have bitten past sessions.
The `README.md` keeps the run/commands/env quickstart and points here.

**AI First CMS** — a conversational, AI-native CMS. Users manage websites through
a chat agent that can read/write pages, run tools, and publish sites, instead of a
traditional admin UI.

---

## Tech stack

- **Monorepo**: Turborepo + npm workspaces. Internal packages use `@ai-first-cms-mvp/*` (`workspace:*`).
- **Frontend** (`apps/web`): Next.js, React 19, TailwindCSS 4, shadcn/ui — port **3051**.
- **API** (`apps/server`): Hono + ORPC (type-safe RPC) — port **3052**, RPC at `/rpc`.
- **Public site** (`apps/public-site`): Astro — renders the published customer sites (port 4000).
- **DB**: PostgreSQL (Docker) via Drizzle ORM. **Two logical databases** — see below.
- **Agent LLM**: OpenAI-compatible provider configured through server environment variables.

## Repo map

```
apps/
├── web/          # Next.js admin dashboard (3051)
├── server/       # Hono + ORPC handler entrypoint (3052) — apps/server/src/index.ts
└── public-site/  # Astro renderer for published sites (blocks + raw-HTML)
packages/
├── api/          # ORPC routers (src/routers/*.ts) + services (llm, mcp, authz, security-events)
│                 #   src/index.ts = the procedure ladder (auth middleware)
├── db/           # Drizzle clients + schema. src/index.ts exports db/dbContent/dbControl
│                 #   src/schema/{content,control,access-tokens}.ts
├── executor/     # Sandboxed side-effect runner (validation, egress allowlist, concurrency)
├── types/        # Shared types incl. src/component-registry.ts (COMPONENT_REGISTRY)
└── env/          # @t3-oss/env-nextjs validation — src/server.ts, src/web.ts
```

## Per-site overlays (`sites/`)

Client/site-specific material (theme, imported assets, seed/content scripts,
docs, and reference files) lives in a separate **private** repo cloned into
`sites/<name>/`, which is gitignored. The CMS repo itself must never contain
client files.

Use `node scripts/site-link.mjs <name>` from the repo root to symlink private
overlay assets into the paths the build expects. Each overlay provides a
`site-link.json` array of `{ "source": "...", "target": "..." }` mappings:
`source` is relative to `sites/<name>/`; `target` is relative to this repo root.
Use `--copy` instead of symlinks for environments that need a copy fallback, and
`--unlink` to remove only links created by the same mapping.

---

## The two-database rule (read this first)

There are **two physical Postgres databases** on one instance, each with its own
role + connection string. They are exported from `packages/db/src/index.ts`:

| Client | Database | Holds |
|--------|----------|-------|
| `dbControl` | `cms_control` | **Crown jewels** — identity, credentials, authorization, audit. |
| `dbContent` (aliased as **`db`**) | `cms_content` | Site-scoped content — everything keyed by a site. |

`db` is a back-compat alias for `dbContent` (the bulk of the app is content-side).
**Control-table callers MUST import `dbControl` explicitly.**

- **Control tables** (`packages/db/src/schema/control.ts`): `organizations`, `users`,
  `orgMembers`, `sessions`, `apiTokens`, `humanActions`, `webauthnCredentials`,
  `webauthnChallenges`, `securityEvents`, `capabilityNonces`, `actionPlans`,
  `actionJobs`, `killSwitches`, `auditAnchors`.
- **Content tables** (`packages/db/src/schema/content.ts`): `sites`, `pages`, `people`,
  `intents`, `pagePeople`, `pageHierarchy`, `pageLinks`, `forms`, `formSubmissions`,
  `jobs`, `pageRevisions`, `siteSnapshots`, `assets`, `pageVectors`,
  `agentConversations`, `agentMessages`, `siteConfig`, `designTokens`, `blockStyles`,
  `stylePresets`, `componentStyles`, `themes`, `siteThemes`, `themeChanges`,
  `contentTypes`, `contentEntries`, `deployments`, `siteHosting`, `redirects`.

**Client-selection constraint:** both clients are constructed with the *full* combined
schema object (only so `.query.X` typings resolve). So `db.select().from(humanActions)`
(a control table on the content client) **compiles fine and fails at runtime** — the
role simply can't see that table. The compiler will not save you today.
→ Always match the client to the table. Cross-DB foreign keys do **not** exist:
`sites.organization_id` and every `*.site_id`/`user_id` on the "other" side is a plain
`text` logical reference, validated at the app tenant guard — never `.references()`.

---

## Auth / procedure ladder

Defined in `packages/api/src/index.ts`. Pick the **most restrictive** one that still
works — never hand a broad procedure a job a scoped one can do.

| Procedure | Guarantees | Use for |
|-----------|-----------|---------|
| `publicProcedure` | none | truly public: `auth.*`, passkey ceremonies, `healthCheck`, `form.submit`. |
| `authedProcedure` | valid session | account-level reads not tied to one site. |
| `siteProcedure` | `siteId` present in context | rare; prefer `authedSiteProcedure`. |
| `authedSiteProcedure` | session **AND** `siteId` **AND** tenant membership | **default for all site-scoped work.** |
| `stepUpProcedure` | authed + fresh (`<5min`) passkey step-up | high-risk management (e.g. domain change). |

`authedSiteProcedure` treats the client-supplied site header as untrusted: the client
supplies `x-site-id`, but the guard derives the user's permitted sites server-side
(`userCanAccessSite`: org memberships in `dbControl` → sites in `dbContent`) and records
a `tenant.access_denied` security event on rejection. Use `hasFreshStepUp()` /
`assertStepUp()` to require step-up **conditionally** inside a handler instead of gating
the whole procedure.

---

## Content model + block system

- **`pages.type`** defaults to `"page"`. **Blogs are `type = "post"`** — not `"blog"`.
  `blog.list` filters `WHERE type='post'`; querying `'blog'` returns an empty blog roll.
  Other types: custom collections live in `contentTypes` / `contentEntries`.
- **`pages.render_mode`** — one of:
  - `"blocks"` (default) — Astro block engine; blocks stored as a jsonb array in `pages.blocks`.
  - `"html"` — full raw-HTML page body in `html_content` (bakes its own chrome).
  - `"article"` — body-only blog post recomposed with the shared blog frame (chrome + CSS).
  - `"framed"` — body-only HTML page recomposed with a **per-site page frame** (the generalised
    version of `article`): one published `template` page (id `tpl_page_frame__<siteId>`, slug
    `/__tpl/page-frame`) holds the chrome + CSS with a `{{CONTENT}}` (and optional `{{MAINCLASS}}`)
    placeholder, and each framed page stores only its body HTML. The renderer recomposes the two
    (`apps/public-site/src/lib/page-frame.ts`), fixing the html-mode chrome-drift problem.
- **Named site components** (`component.define_html` / `component.list_html`): the middle
  rung of the promotion ladder (raw HTML → **named per-site template** → first-party block).
  A component is an HTML template defined ONCE and reused, parameterized, across pages. Storage
  mirrors the page/blog frame convention: a `pages` row with id `tpl_component_<name>__<siteId>`,
  slug `/__tpl/component/<name>`, `type='template'`, and `status='published'` (published — NOT
  draft — because the content loader + live-snapshot builds both filter `status='published'`, so
  a draft template would never render; route/sitemap/`page.list` exclusion via the `/__tpl/` slug
  + `type='template'` is what keeps it from shipping as its own page). The template body uses a
  mustache-lite syntax — `{{key}}` (HTML-escaped), `{{{key}}}` (raw), `{{#each items}}…{{/each}}`
  (one level, `{{field}}` + `{{@index}}`); missing keys render empty. Instantiate it in any
  html/article/framed page body with a marker `<div data-component="name" data-props='{…}'></div>`
  (`<section>` also works); at render the whole marker is replaced by the expanded template (unknown
  name / invalid JSON → left untouched). Expansion runs BEFORE `sanitizePageHtml` at all four render
  paths (`apps/public-site/src/lib/site-components.ts`), so templates get NO script privileges.
  Redefining a name restyles every instance sitewide.
- **Block registry**: `packages/types/src/component-registry.ts` — `COMPONENT_REGISTRY`
  is the catalog the admin browser and the AI agent read to discover blocks + default
  props. Each block's rendered shape lives in `apps/public-site/src/components/**`
  (dispatched by `BlockRenderer.astro`). Block payload validation on write is in the
  executor (`packages/executor/src/validation.ts`) and `page.create`/`page.patch`.
  Block payloads must be validated on every write before reaching the renderer.
- **`seo` jsonb** on `pages` is a partial junk drawer. Legacy keys: `title`, `description`,
  `featured_image_id`, `reading_time`, `markdown`. **SEO v2 head keys** (consumed by
  `apps/public-site/src/layouts/BaseLayout.astro`): `canonical`, `og_title`, `og_description`,
  `og_image`, `noindex` (per-page opt-in, in addition to the type/slug rule in
  `apps/public-site/src/lib/noindex.ts`), and `json_ld` (an arbitrary object emitted verbatim as
  an extra `<script type="application/ld+json">`). `source_markdown` is the real markdown column
  (KB articles, served raw at `<slug>.md`).

### Staging → Live isolation (snapshots)

Snapshots isolate editable staging content from the content currently served as Live:

- `siteSnapshots` = immutable, gzipped, full-site content captures (every page incl.
  `html_content`, redirects, config, tokens, theme, people).
- `sites.live_snapshot_id` points at the snapshot currently served to **Live**. The live
  build sources content from that snapshot, **not** the working `pages` rows. Null until
  the first Publish-to-Live; the public loader falls back to working rows while null.
- Publishing writes a new `publish`-kind snapshot and repoints `live_snapshot_id`.
- **Fail-closed contract:** live builds select the live snapshot content source
  next to `DEPLOY_SNAPSHOT_FILE`; the public-site content loader then hard-fails the build if
  the snapshot file is missing/unreadable/malformed instead of silently falling back to
  working DB rows. Unset var = legacy staging/dev behavior.

---

## Agent tool system

The chat agent (`packages/api/src/routers/chat.ts`) runs a multi-step loop:
`POST /rpc` → `chat.sendMessage` → `callAgent()` in `services/llm.ts`
→ tool calls → `executeToolCall()` feeds results back until a final answer.

Tools (definitions in `llm.ts`, execution in `chat.ts`):
- **Pages**: `page.get`, `page.list`, `page.create`, `page.patch`, `page.set_html`,
  `page.patch_html` (edit tagged `data-edit-field` regions), `page.edit_html` (str_replace-style
  edit of raw-HTML/article bodies), `page.publish`, `page.screenshot` (render → multimodal critique).
- **Blog**: `blog.create_post`, `blog.update_post`, `blog.list_posts` (posts are `pages` rows with
  `type='post'`; markdown→blocks lives in `packages/validator/src/markdown.ts`).
- **Content types**: `content.define_type`, `content.list_types`, `content.create_entry`,
  `content.list_entries`, `content.update_entry`.
- **Other**: `component.list` (reads `COMPONENT_REGISTRY`), `component.define_html` /
  `component.list_html` (named site components — see the content-model section), `asset.upload_from_url`
  (SSRF-guarded external image → self-hosted same-origin under `public/_ext/<siteId>/`), `theme.set_tokens`.

The dangerous host-access tools (`code.read` / `code.write` / `terminal.run`) were **removed** from
both the agent tool surface AND the executor — the executor's `ALLOWED_INTENTS` set is the final
enforcement boundary and must never re-admit them without a security review. Advertised tools must
also remain limited to the capabilities granted for the current agent session.

**External MCP surface** (`packages/api/src/mcp/`): a JSON-RPC MCP server (`server.ts`) exposes a
subset of the above to external agents (e.g. Claude Code) behind per-tenant **PAT** auth. Every tool
is bound to the token's site (never a client-named site → `CROSS_SITE`) and routes writes through the
SAME `createIntent`→`executeIntent` path as chat. Two scopes: **`content:read`** (`page.get`,
`page.list`, `component.list`, `component.list_html`, `blog.list_posts`, `content.list_entries`) and
**`content:write`** (`page.create`, `page.set_html`, `page.edit_html`, `page.patch_html`, `page.patch`,
`page.publish`, `blog.create_post`, `blog.update_post`, `component.define_html`, `theme.set_tokens`,
`asset.upload_from_url`). Add a new MCP
tool in `mcp/tools.ts` following the existing structure — declare its scope, keep the PAT binding.

Side effects are modeled as **intents** (`intents` table: idempotency key, risk level,
rollback ref) and, for multi-site batches, as `actionPlans` → `actionJobs` (manifest-hash
approval, canary, second approver for critical/>50-site plans). `killSwitches` freeze a
capability (`global:agent`, `global:publish`, `site:<id>:publish`, …) — checked before
agent runs, publishes, and capability minting.

---

## Deploy pipeline (of a customer site)

A site publish builds the Astro `public-site` and ships static output to a hosting
provider. State is tracked in the `deployments` table.

- **`deployments.status` lifecycle**: `pending → building → uploading → live` (or `failed`).
- **`deployments.provider`**: `local` | `cloudflare-pages` | `bunny`
  (null legacy rows = `local`). Per-site choice lives in `siteHosting`
  (provider + region + non-secret `project_name`/zone refs). **Provider credentials are
  NEVER stored in the DB** — they are account-level in server env
  (`CLOUDFLARE_*`, `BUNNY_API_KEY`).
- **Hosted subdomain**: when a platform domain and hosting provider are configured, a
  publish can attach a project subdomain using the provider's documented DNS flow.
- **Deploy-row gotcha**: a row stuck in `building`/`uploading` blocks the next deploy
  (retries throw `DEPLOY_IN_PROGRESS`). A reaper clears stale rows; if a deploy is wedged,
  check for and clear the stuck row before retrying.

---

## Database schema migrations (generated, NOT `push --force`)

Schema is applied via **generated Drizzle migrations** (`drizzle-kit migrate`), not
`push --force`. `push --force` has no history and silently drops/rewrites on any rename —
a data-loss hazard. The server entrypoint (`docker/server-entrypoint.sh`) runs
`db:baseline` then `db:migrate:control` + `db:migrate:content` on every boot (idempotent).

- **Two migration chains, two journals**: control migrations live in
  `packages/db/src/migrations/control/` (journal table `drizzle.__drizzle_migrations_control`),
  content in `packages/db/src/migrations/` (`drizzle.__drizzle_migrations_content`). The
  per-DB journal-table name (set via `migrations.table` in each `drizzle.config.*.ts`) is
  what lets both chains run against the SAME physical DB in CI without colliding.
- **Changing the schema** (the flow to follow):
  1. Edit `packages/db/src/schema/{content,control,access-tokens}.ts`.
  2. `npm run -w @ai-first-cms-mvp/db db:generate:content` (or `:control`) — writes a new
     `NNNN_*.sql` + snapshot. **Commit the generated files.**
  3. `npm run -w @ai-first-cms-mvp/db db:migrate:content` (or `:control`) locally to apply.
  4. Deploy — the entrypoint applies the committed migration on prod boot.
- **`db:baseline`** (`scripts/baseline-migrations.mjs`): one-time cutover helper. An
  existing push-built DB has all tables but no journal, so baseline marks migration `0000`
  as already-applied WITHOUT re-running its `CREATE TABLE`s (which would error). A fresh DB
  has no tables + no journal, so baseline no-ops and `migrate` creates everything from 0000.
  Idempotent: once the journal has rows, baseline does nothing.
- `db:push` / `db:push:*` still exist for throwaway local scratch DBs, but the source of
  truth is the committed migration files — do NOT push against a DB you then migrate.

---

## Error convention

Routers use `ORPCError(code, {message})` (`UNAUTHORIZED`, `FORBIDDEN`, `BAD_REQUEST`,
`NOT_FOUND`, …). Error responses must contain safe client-facing messages only; log
full causes server-side and never return stacks or database-driver details.

---

## Environment (see `packages/env/src/{server,web}.ts` for the source of truth)

Required in root `.env`:

```env
DATABASE_URL_CONTROL=postgresql://.../cms_control   # crown-jewels DB
DATABASE_URL_CONTENT=postgresql://.../cms_content   # site content DB
# DATABASE_URL=...  (legacy single-DB; optional fallback only)
CORS_ORIGIN=http://localhost:3051
API_TOKEN=test_token_12345          # must be overridden in prod
NEXT_PUBLIC_SERVER_URL=http://localhost:3052
OPENROUTER_API_KEY=replace-me       # example OpenAI-compatible provider credential
```

`apps/web/.env` needs `NEXT_PUBLIC_SERVER_URL=http://localhost:3052`.
Optional: `UMAMI_*` (analytics), `CLOUDFLARE_*` / `BUNNY_API_KEY` (hosting),
`PLATFORM_HOSTED_DOMAIN` (free subdomains),
`RUNWARE_API_KEY` (image gen). A provider is only offered in the UI when its
credentials are present.

---

## Local dev

```bash
npm install
docker-compose up -d       # Postgres (Docker)
# Apply schema via generated migrations (prod uses the same path). db:baseline is
# a no-op on a fresh DB; on an existing push-built DB it marks 0000 applied first.
npm run -w @ai-first-cms-mvp/db db:baseline
npm run -w @ai-first-cms-mvp/db db:migrate:control
npm run -w @ai-first-cms-mvp/db db:migrate:content
# seed — needs the ALLOW_CONTENT_ADMIN=1 gate, and turbo strips ad-hoc env vars,
# so run the seed directly rather than through `npm run db:seed`:
(cd packages/db && ALLOW_CONTENT_ADMIN=1 npx tsx src/seed.ts)
npm run dev                # web:3051 + server:3052
npm run check-types        # typecheck all packages (the CI/commit gate)
```

Adding an endpoint: add a procedure in `packages/api/src/routers/*.ts`, register it in
`routers/index.ts`; frontend types are auto-inferred via the ORPC client
(`apps/web/src/utils/orpc.ts`). When invalidating a query cache, use
`orpc.X.queryKey()` (flat-query-key gotcha).

---

## Use-this-not-that

| Doing this | Use this, not that |
|------------|--------------------|
| Reading a block's current content | Read from **`data`**, not stale component props (commit `c843640`). |
| Writing a control-side row (auth/audit/creds) | `dbControl.*`, never `db`/`dbContent`. |
| A site-scoped procedure | `authedSiteProcedure` (tenant-checked), not `publicProcedure` + manual header trust. |
| Listing blog posts | `WHERE type = 'post'`, not `'blog'`. |
| Serving Live content | Read the site's `live_snapshot_id` snapshot, not working `pages` rows. |
| Storing provider creds | Account-level server env, never the `siteHosting`/`deployments` row. |
| Checking this app is up | Use the health endpoint configured for your deployment. |
| Invalidating an ORPC query | `orpc.X.queryKey()`, not a hand-built key. |
| Generating any image (design.concept / design.improve / `/api/images/fallback`) | Use the configured image model and quality setting consistently. Defaults are enforced in `packages/api/src/services/image-generation.ts` + `apps/server/src/lib/image-fallback.ts`. |

---

## Keeping this file current

Update this document whenever the architecture or operational contracts change so it
remains the single source of truth for contributors.

---
> Source: [Marc-Moeller/aicms](https://github.com/Marc-Moeller/aicms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
