## hayloft

> Self-hosted archive for Harvest time-tracking data: a rate-limited fetcher

# Hayloft

Self-hosted archive for Harvest time-tracking data: a rate-limited fetcher
(`src/fetcher/`) pulls everything into Postgres (`src/db/schema.sql`), and a
Hono JSX SSR app (`src/web/`) serves it read-only behind a shared password.

- `npm run typecheck` must pass before committing.
- Local dev: `docker compose up -d`, `npm run migrate`, `npm run seed-demo`
  (fake data, no Harvest account needed), `npm run dev`.
- SSR only — no client-side framework. Reuse the DB helpers in
  `src/lib/db.ts` (`q`, `qOne`, `qScalar`, `exec`).
- New Harvest resources follow the existing pattern: entry in
  `src/fetcher/resources.ts` + table in `src/db/schema.sql` with parsed
  columns plus a `raw_json` escape hatch.

## Task tracking

Hayloft's work is tracked in `TASKS.md` at the repo root. Keep it current as you work:
- Check off (`- [x]`) tasks you complete and let them move to `## Done`.
- Move tasks between `## In Progress`, `## Next`, and `## Backlog` as priorities change.
- Add newly discovered work as `- [ ]` items under the right section.
- Preserve each task's `^id` tag — it is a stable identifier used to track the task across edits.

---
> Source: [WebDevStudios/Hayloft](https://github.com/WebDevStudios/Hayloft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
