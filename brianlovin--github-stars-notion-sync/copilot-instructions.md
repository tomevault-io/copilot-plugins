## github-stars-notion-sync

> Notion Worker that syncs a user's GitHub starred repos into a Notion database, plus local CLI scripts for setup, full-history backfill, and dedup. Also exposes `star_repo` / `unstar_repo` tools that a Notion AI agent can invoke.

# github-stars-notion-sync

Notion Worker that syncs a user's GitHub starred repos into a Notion database, plus local CLI scripts for setup, full-history backfill, and dedup. Also exposes `star_repo` / `unstar_repo` tools that a Notion AI agent can invoke.

See [README.md](./README.md) for user-facing docs.

## Architecture (one line)

`scripts/setup.ts` creates a user-owned Notion database via direct REST API; `scripts/backfill.ts` walks every starred repo via the GitHub API and upserts rows via direct Notion API; `src/index.ts` is a thin daily delta worker (plus two `worker.tool` capabilities for star/unstar).

## Rules

- [Conventions](./.claude/rules/conventions.md) — TypeScript, file layout, Notion API version pin
- [Invariants](./.claude/rules/invariants.md) — what setup owns vs the worker; auth scopes; dedup
- [Common tasks](./.claude/rules/tasks.md) — add a property, change schedule, rotate OAuth, redo backfill, add a tool
- [Workflow](./.claude/rules/workflow.md) — plan first, verify, capture lessons

## Verification

- `npm run check` — type-check (no emit)
- `npm test` — unit tests against fixtures (~200ms, no network)

No E2E test. Functional verification needs a live deploy + `ntn workers runs logs <id>`.

## Don't

- **Don't route data-DB writes through the worker's managed-sync `changes[]`.** That path can't host markdown bodies, rejects open-vocab selects, and the framework migrates data sources on schema changes. The user-owned DB + direct REST API path is deliberate; see [`tasks/lessons.md`](./tasks/lessons.md) for the receipts.
- **Don't bypass the GitHub pacer** — secondary rate limits are silent and punitive.
- **Don't add per-repo `console.log` inside the worker's hot loop.** Counters go into the audit row body; `console.error` only for actual failures.
- **Don't hardcode user-specific values** (database IDs, OAuth client IDs) in `src/` or `scripts/` — env vars or runtime-resolved.
- **Don't commit `.env` or `workers.json`.** Both are git-ignored.

---
> Source: [brianlovin/github-stars-notion-sync](https://github.com/brianlovin/github-stars-notion-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
