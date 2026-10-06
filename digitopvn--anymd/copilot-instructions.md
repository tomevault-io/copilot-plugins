## anymd

> Guide for AI coding agents working in this repository. anymd.cc converts any URL to clean Markdown and keeps a searchable per-user library, exposed through the URL API, REST, CLI, MCP and WebMCP. Owner: Digitop.ai (hello@digitop.ai). License: MIT.

# AGENTS.md

Guide for AI coding agents working in this repository. anymd.cc converts any URL to clean Markdown and keeps a searchable per-user library, exposed through the URL API, REST, CLI, MCP and WebMCP. Owner: Digitop.ai (hello@digitop.ai). License: MIT.

## Stack

- One Cloudflare Worker: Hono (routing + JSX SSR), TypeScript, zod.
- Cloudflare D1 (with FTS5), KV, R2, Vectorize, Workers AI, rate-limit bindings.
- Conversion: the `src/convert/web.ts` extractor + linkedom, turndown; Workers AI `toMarkdown` for files.
- Tailwind CSS v4; client islands bundled with esbuild; vitest for tests.
- MCP: `@modelcontextprotocol/sdk`, OAuth via `@cloudflare/workers-oauth-provider`. Billing: Polar.sh on production, Creem.io on staging, picked by `BILLING_PROVIDER`.

Bindings, vars and secrets are typed in `src/env.ts`; per-environment values in `wrangler.jsonc`.

## Commands

From `package.json`:

| Command | Does |
|---|---|
| `npm run build` | Build CSS (`build:css`) and client JS (`build:js`) |
| `npm run dev` | Build, then `wrangler dev --env staging` |
| `npm run typecheck` | `tsc --noEmit` |
| `npm test` | `vitest run` |
| `npm run db:migrate:staging` / `db:migrate:production` | Apply D1 migrations remotely |
| `npm run deploy:staging` / `deploy:production` | Build and `wrangler deploy` |

Before handing work back: `npm run typecheck`, plus `npm test` for touched behavior. Deploys and remote migrations need explicit permission from the user.

## Where things live

- `src/convert/`: SSRF guard, adapter registry, the single conversion pipeline, source adapters.
- `src/library/`: library storage, embeddings, search, Jev.
- `src/auth/`: principal resolution, scope guards, roles and key presets.
- `src/billing/`: plans, credits, offers, the active provider (`provider.ts`), Creem and Polar.
- `src/cms/`: page-builder blocks and page ops service.
- `src/lib/`: usage/quota, tracer, Markdown rendering, email, utilities.
- `src/content/`: all bundled Markdown content and site copy.
- `src/views/`, `src/routes/`, `src/mcp/`, `client/`, `cli/`: UI, HTTP routes, MCP server, browser islands, CLI.
- `migrations/`: D1 schema. `docs/`: maintainer docs.

Start with `docs/architecture.md` for the module map and request flow, `docs/operations.md` for deploys, logs and rollback, and `REVIEW.md` for what reviewers check.

## Conventions

- **One pipeline.** Every conversion goes through `runConversion` in `src/convert/service.ts`. Never fetch a user-supplied URL outside `normalizeTargetUrl`.
- **Scopes on every route.** Guard API routes with `requireScope`; scope every library query by `user_id`; hide MCP tools the caller can't use.
- **Side effects in `waitUntil`.** Usage recording, embeddings and billing ingest must not add latency or fail the request.
- **Errors** are `{ error: { code, message, ... } }` with a matching status. Clients branch on `code`.
- **Public contracts** (URL API, REST, MCP tools, WebMCP tools, CLI, credits) are documented in `src/content/docs/`. Change code and the matching doc page together.
- **Optional secrets.** New integrations must detect their secret and degrade gracefully without it.
- **Content truth.** Prices and credits come from `src/billing/plans.ts`. No invented metrics, testimonials or logos anywhere. Roadmap sources in `ROADMAP_SOURCES` (`src/content/site.ts`) are "planned", never described as shipped.
- **Every public page has a `.md` twin.** New pages and blocks must produce good Markdown.
- Keep files focused; follow existing patterns before adding abstractions. Conventional commits; no secrets in commits.

## Secrets

- Never print, log, echo, commit or paste values from `.env`, `.dev.vars` or `wrangler secret`. Refer to secrets by name only.
- Don't `cat` env files. If you need to know which secrets exist, read `src/env.ts` or run `npx wrangler secret list --env <env>` (names only).
- Never put real keys in docs, examples, tests or fixtures. Use `amd_…` placeholders.

## Content

All site copy is Markdown under `src/content/`, imported as text by Wrangler and parsed by `src/content/index.ts`:

- `blog/*.md`: frontmatter `title`, `slug`, `excerpt`, `category`, `tags`, `published_at`, `seo_title`, `seo_description`.
- `docs/*.md`: frontmatter `title`, `description`, `updated`; body without a leading H1 (the template renders the title). Cross-link as `/docs/<slug>`.
- `legal/*.md`: legal pages.
- `site.ts`: site facts, navigation, sources, roadmap, ecosystem.

## How to

### Add a doc page

1. Create `src/content/docs/<slug>.md` with the frontmatter above.
2. Import it in `src/content/index.ts` and add `contentPage('<slug>', …)` to `DOCS_PAGES` at the right position (array order is navigation order).
3. Link it from `src/content/docs/index.md`.

### Add a page-builder block

1. Add a `define({...})` entry to `BLOCKS` in `src/cms/blocks.ts`: `type`, `version`, `label`, `description`, `category`, zod `schema` with length/count limits, `sizes`, `defaultSize`, optional `slots`, a valid `example`, and `toMarkdown`.
2. Add its renderer in the render module named in the `blocks.ts` header.
3. The catalog (`GET /api/v1/admin/blocks`, MCP `list_blocks`) picks it up automatically. Add a row to the block table in `src/content/docs/page-builder.md` (served to admins at `/admin/docs/page-builder`, not in public docs).
4. Changing an existing block's props shape: bump `version` and keep old documents rendering.

### Add a source adapter

1. Implement `SourceAdapter` (`kind`, cheap `matches(url)`, `convert(url, ctx)`) in `src/convert/<source>.ts`. Use `ConvertError` with a stable `code`; use `ctx.tracer.span` around network calls.
2. Add a new `SourceKind` in `src/convert/types.ts` if needed.
3. Register it in `ADAPTERS` in `src/convert/index.ts`, before `webAdapter` (order matters).
4. Set its cost in `creditCost` / `documentCreditCost` and `CREDIT_TABLE` (`src/billing/plans.ts`).
5. Add it to `SOURCES` in `src/content/site.ts` (and remove it from `ROADMAP_SOURCES` if it was planned), then update `src/content/docs/sources.md` and `billing.md`.
6. Any third-party API key goes in `src/env.ts` as an optional secret.

---
> Source: [digitopvn/anymd](https://github.com/digitopvn/anymd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
