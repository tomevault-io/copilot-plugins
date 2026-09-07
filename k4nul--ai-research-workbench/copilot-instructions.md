## ai-research-workbench

> These rules supplement the workspace-level `AGENTS.md` for work in this repository.

# Repository instructions

These rules supplement the workspace-level `AGENTS.md` for work in this repository.

## Scope and architecture

- Use Node.js 22 or newer and npm.
- Keep route handlers in `app/api`, orchestration and database work in `lib/services`, pure rules in `lib/domain`, provider adapters in `lib/providers`, security boundaries in `lib/security`, and rendering in `lib/export`.
- PostgreSQL is accessed through `pg`; schema changes are additive raw SQL migrations under `migrations/`. Do not introduce an ORM without an explicit architecture decision.
- Preserve the source → evidence → claim → finding → deliverable graph and append audit events for state-changing workflow actions.
- Mock providers are the default. Never make tests depend on provider keys or live network access.
- The PostgreSQL `jobs` table and separate `DurableWorker` process implement at-least-once delivery. Preserve DB-time leases, heartbeats, attempt fencing, cooperative cancellation, bounded retry, idempotent external effects, and stage/domain commit keys; never describe this as exactly-once execution.
- Keep uploaded, quarantine, promoted clean-source, extraction, evaluation, and export artifacts in the private local/S3 storage abstraction with cataloged size/SHA-256 metadata and compensating cleanup.

## Commands

```bash
npm ci
npm run db:start
npm run db:migrate
npm run seed
npm run dev
npm run worker
npm run doctor
npm run eval:mock
```

`ALLOW_DATABASE_RESET=true npm run db:reset` is destructive. Run it only against a confirmed disposable local database after checking `DATABASE_URL`. Database tests must use `TEST_DATABASE_URL`; its database name must contain `test`.

## Change and test rules

- Inspect the nearest route, service, domain rule, and test before editing.
- Make the smallest change that proves the requested behavior; do not refactor unrelated code.
- Add focused Vitest coverage for domain, validation, provider, security, or service behavior.
- Provider tests must mock `fetch` and validate source-ID allowlists and structured output.
- Security changes require negative cases for hostile URLs, redirects, DNS results, filenames, MIME/signatures, size bounds, or untrusted HTML as applicable.
- QA blockers are strict: only `RESOLVED` clears a blocker. `ACCEPTED_RISK` is non-blocking only for non-blocker severities.
- UI behavior requires Playwright or equivalent browser evidence; component or file existence is not sufficient.
- Export changes require content tests plus opening generated PDF/DOCX artifacts and inspecting ZIP entries when practical.
- Keep sample data visibly synthetic. Never add customer data, secrets, private URLs, or real provider responses to fixtures.
- Update the relevant file under `docs/` when a workflow, schema, provider, security boundary, QA rule, or export contract changes.

Run the narrowest relevant command first, then the full applicable gate:

```bash
npm run test:unit
npm test
npm run typecheck
npm run lint
npm run build
npm run test:e2e
npm run verify
```

Do not report a configured but empty test lane as passing behavior-level verification.

## Security and compatibility

- Treat fetched, uploaded, and model-produced text as untrusted input.
- Preserve SSRF protections across every redirect and DNS resolution; do not weaken byte/time/content-type bounds.
- Keep storage outside public roots and preserve containment checks and restrictive file permissions.
- Do not add a public deployment until authentication, authorization, CSRF, TLS, secrets, rate limiting, tenancy, backup, and retention decisions are explicit.
- Avoid public API or migration-contract changes without approval.

## Commits

Do not commit unless requested. Use one logical purpose per commit and this format:

```text
type(scope): subject
```

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`, `rename`, `remove`. The scope must match `[a-z0-9][a-z0-9-]*`; keep the subject concise and lowercase.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [k4nul/ai-research-workbench](https://github.com/k4nul/ai-research-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
