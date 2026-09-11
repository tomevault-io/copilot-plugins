## anixd-m

> A code-architecture tour aimed at AI coding agents and new contributors. Read this **before** editing anything in `src/lib/`, `src/app/api/`, or any `ddl-*.sql` file.

# AGENTS.md

A code-architecture tour aimed at AI coding agents and new contributors. Read this **before** editing anything in `src/lib/`, `src/app/api/`, or any `ddl-*.sql` file.

## Project at a glance

**ANIXD-MARKETING** — a self-hostable multi-channel marketing platform. Next.js 16 (App Router) + React 19 + TypeScript + PostgreSQL 14+. The codebase was extracted from a production deployment, so you'll see mature code with edges — assume any file you open has been load-tested at real scale.

## Repo layout

```
app/                 Next.js App Router pages and route handlers
  (admin)/           Auth-gated admin pages
  api/               Route handlers (REST-ish)
src/lib/             Core domain logic — most non-trivial code lives here
  channels/          Per-channel integrations (email, sms, whatsapp, rcs)
  automation/        Automation engine (rules, steps, conditions)
  db/                Postgres connection helpers
  campaign-engine.ts The campaign send queue (drain, retry, status mapping)
  flow-engine.ts     Chat flow evaluation (used by chat-builder + receipt flow)
  short-links.ts     Branded short-link resolution
  receipt-pdf.ts     PDF receipt generator
  engagement.ts      Per-contact rollup
src/ai/              LLM prompt templates and tool definitions
src/components/      React components (shadcn/ui + Radix)
tests/               Vitest test suites (mirrors src/ structure)
scripts/             Operational scripts (cron, watchdog, doctor, deploy-verify)
ddl-*.sql            Postgres schemas (idempotent — safe to re-apply)
backfill-*.sql       One-off data migrations
public/              Static assets (PDF fonts only — no logos)
```

## Conventions

### Imports

- Use the path alias `@/` for everything under `src/` and `app/`. **Never** use relative `../../..` — it gets ugly fast in this repo.
- `import type { X }` for types; the project is `strict` and `noUncheckedIndexedAccess` is on.
- Prefer named exports. Default exports are reserved for Next.js page/layout components.

### Database

- **Every table row has `org_id`** for multi-tenant scoping. Filter on it in every query, including counts and aggregates.
- Use the helpers in `src/lib/db/`. Don't open a new `pg.Pool` per request.
- DDL lives in `ddl-*.sql` files. They are **idempotent** (`CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`) so they can be re-run on every deploy.
- The `campaign_messages` queue is the single source of truth for "what should be sent". Never write to a provider without first writing a `campaign_messages` row.

### Channels

- Each channel has a folder under `src/lib/channels/` and a route under `app/api/channels/<channel>/`. Both share a `send()` shape: `{ success, providerMessageId, error }`.
- **Provider status mapping** lives in `src/lib/channels/<channel>-status.ts`. The `provider_response` table is the audit log — every API call records its raw response there.
- Rate limits per provider are defined in `src/lib/provider-limits.ts`. The campaign engine consults them per send.

### Automations

- An automation = `automation_rules` row + N `automation_steps` rows.
- Steps form a DAG evaluated by the engine in `src/lib/automation/`.
- The cron endpoint `app/api/cron/evaluate-automations/route.ts` runs every active rule.
- Per-contact per-rule cooldown lives in `automations_engagement` — never send the same automation twice to the same contact inside the cooldown window.

### AI / LLM

- The audience workbench (`src/ai/`) and automation planner use a shared LLM client (`src/ai/llm-client.ts`) that respects a 30s cache (set `LLM_CACHE=0` to disable).
- Prompts are large and domain-specific. **Don't rewrite them from scratch** — add a section, don't replace the existing one.
- Every LLM call has a "raw" vs "parsed" path; the raw path is logged to `provider_response` for debugging.

### Auth

- `src/middleware.ts` gates `/admin` and `/api/admin`.
- The most sensitive admin routes additionally check `ADMIN_USER`/`ADMIN_PASS` against the session.
- Public-facing routes (e.g. `/l/[code]`, `/login`) bypass the middleware.

### Testing

- Vitest + Testing Library. Tests live in `tests/` and mirror the `src/` structure.
- `tests/setup.ts` sets the env vars you need for the test environment.
- Run `npm test` (one-shot) or `npm run test:watch` (watch).
- Coverage: `npm run test:coverage` — output goes to `./coverage/`.
- New features need tests. Bug fixes need a regression test that fails without the fix.

### Linting

- `npm run lint` runs `eslint` with `eslint-config-next`. The config is in `eslint.config.mjs`.
- Don't add eslint-disable comments. If a rule fires, fix the code or open a discussion.

## Where to start (depending on the task)

| Task | Start here |
|---|---|
| Add a new channel (e.g. push notifications) | `src/lib/channels/` + `app/api/channels/<channel>/` + `src/lib/provider-limits.ts` + a `ddl-<channel>.sql` |
| Add a new automation trigger | `src/lib/automation/triggers/` + `automation_rules` DDL |
| Add a new field to contacts | `contacts` DDL + a backfill SQL in `backfill-*.sql` + update `src/lib/contact-fields.ts` |
| Add a new admin page | `app/(admin)/<route>/page.tsx` + a sidebar entry in `src/components/layout/sidebar.tsx` |
| Add a new API route | `app/api/<path>/route.ts` — pick `GET`/`POST` exports; check auth in `src/middleware.ts` |
| Add a new cron job | `app/api/cron/<job>/route.ts` + add to `scripts/install-cron.sh` |

## Common gotchas

- **`useChat` / streaming routes** — there are a few; they MUST stay server-side because of provider auth keys. Don't accidentally `use client` them.
- **PDF generation** uses pdfkit and reads AFM font files from `public/fonts/`. Don't delete that folder. The receipt generator (`src/lib/receipt-pdf.ts`) is the only consumer.
- **Time math** — all timestamps are stored as UTC `timestamptz`. Don't store local times anywhere.
- **`noUncheckedIndexedAccess`** is on. `array[i]` returns `T | undefined`. Code accordingly.
- **Provider webhooks** must respond `200` even on bad payloads, otherwise providers retry forever and poison the log. See `src/app/api/webhooks/*/route.ts` for the pattern.
- **Short-link redirects** (`app/api/l/[code]/route.ts`) are tracked — every redirect writes a `short_link_clicks` row. Don't optimize it without checking the indexing on that table.

## Things to avoid

- **Don't add new dependencies lightly.** Many of the deps are pinned to specific versions for a reason. Open an issue first if you think a new dep is required.
- **Don't merge migrations into existing `ddl-*.sql` files.** Add a new `ddl-<feature>.sql` instead. This keeps `git log --follow` clean and lets you re-apply schemas independently.
- **Don't add `any`.** The codebase is `strict`. Use `unknown` + a Zod parse if the type really is unknown.
- **Don't read from `process.env` outside `src/lib/config.ts`** (if it exists) or the integration files in `src/lib/channels/`. Otherwise env-var mocking in tests breaks.
- **Don't rename `org_id` columns.** Every query in the codebase filters on it.
- **Don't write to `provider_response` directly.** Use the helper in `src/lib/provider-log.ts` (if it exists) — that helper handles the "did the call actually succeed?" audit question correctly.

## Pull request checklist

- [ ] Tests added or updated
- [ ] `npm test` passes
- [ ] `npm run lint` passes
- [ ] If schema changed: new `ddl-*.sql` file added (don't edit existing)
- [ ] If env var added: documented in `.env.example` with a comment
- [ ] If new provider integrated: stubbed keys in `.env.example`, real keys documented in README
- [ ] PR description explains the **why**, not just the what

## Questions?

Open an issue. Tag it `question` and reference the file/area you're asking about.

---
> Source: [aniflax/ANIXD-M](https://github.com/aniflax/ANIXD-M) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
