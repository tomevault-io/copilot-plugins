## studio-playlists

> Instructions for coding agents working in this repository.

# AGENTS.md — studio-playlists

Instructions for coding agents working in this repository.

## What this repo is

Flask API + Postgres schema for YouTube playlist management. Owns playlist identity,
ordered membership, and publishing state. References videos owned by studio-videos —
never their truth. See `MEMORY.md` for decisions, `TECH-SPEC.md` for schema/API,
`TODO.md` for backlog.

## Boundaries (do not violate)

- **Never store video production facts here.** Only `video_ref_id` + `cached_*` display
  fields. If a feature needs more video data, extend the studio-videos snapshot contract
  (coordinate there), don't widen local columns.
- `video_ref_id` is not a DB foreign key and must stay that way — integrity is
  contract-level via the snapshot API; reads must never require studio-videos to be up.
- Placeholders (`video_ref_id NULL`) are first-class; no feature may assume every item is
  linked.
- No course concepts; no generic `content_type`/`parent_id` patterns.
- Todos, audit, settings, Todoist, observability come from `studio-core` — never
  reimplemented here.

## Commands (container-first)

```bash
docker compose up --build                                   # api :5002, db :5434
docker compose exec -T api uv run pytest -q
docker compose --profile test run --rm integration-tests
docker compose exec -T api uv run alembic upgrade head
docker compose exec -T api uv run alembic revision --autogenerate -m "<msg>"
docker compose exec -T api uv run ruff check .
```

## Hard rules

1. Every schema change is an Alembic migration.
2. Every route is org-scoped; every mutation records an audit event.
3. Reorder is atomic — one transaction, deferrable unique constraint; never best-effort
   row-by-row position updates.
4. `videos_client` failures degrade (stale cache + report), never 500 a playlist read.
5. Vendored snapshot-contract fixtures must match the pinned studio-videos version; update
   both together.
6. Regenerate `openapi.json` on contract changes; routes ship with tests in the same PR.

## Session workflow

- Start: read `MEMORY.md` and `TODO.md`. End: update both (decisions dated, with why) and
  `CHANGELOG.md` for user-visible changes.
- Commits: imperative subject, body explains why.

## Definition of done

Code + migration + tests green in compose + openapi.json regenerated + TODO/MEMORY/CHANGELOG
updated.

---
> Source: [awesomejt/studio-playlists](https://github.com/awesomejt/studio-playlists) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
