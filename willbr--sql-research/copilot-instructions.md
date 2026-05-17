## sql-research

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running and iterating

- `uv run sql-learn` or `uv run app.py` — start the Flask dev server on port **5001** (not 5000, because macOS AirPlay binds 5000).
- `rm learn.db` — force a fresh re-seed from `schema.sql` on next app start. The web UI's **Reset database** button does the same thing without a restart.
- `lsof -ti:5001 | xargs kill` — if a lesson edit + restart loop leaves a stale server bound to 5001, the new process fails silently and serves an empty DB. Kill the old one and retry.

There is no test suite. For query work, verify changes by starting the server and `curl --data-urlencode "query=…" http://127.0.0.1:5001/`.

## Architecture

Single-file Flask app (`app.py`) + one Jinja template. The moving parts worth reading together:

### Lessons are a Python list, not a config file

`LESSONS` in `app.py` is the single source of truth for the sidebar. Each item has `title`, `prompt`, `example`. The template renders them as clickable cards; clicking one dispatches a CodeMirror change that replaces editor contents. To add a lesson, append to the list — no other files need editing.

### Query execution: `execute()` vs `executescript()`

The `index` route decides which to call based on whether the trimmed query body contains a `;` before its final semicolon (heuristic multi-statement detection). `execute()` runs one statement and can return rows for the results table; `executescript()` runs many but returns no row data. This is why lessons 12 (DROP chain) and 28 (CTE + DELETE) run but don't render a result table unless the final statement has a `RETURNING` and there's only one statement.

### DB lifecycle + SQLite default overrides

`create_app()` calls `init_db()` iff `learn.db` is missing. `/reset` always calls it (drops + recreates from `schema.sql`). Per-request connections in `get_db()` apply two non-default settings that the lessons depend on:

- `isolation_level=None` — autocommit, so DDL *and* DML persist after a single `execute()`.
- `PRAGMA foreign_keys = ON` — SQLite ships with FK enforcement off; turning it on per-connection is what makes the DML lessons hit `FOREIGN KEY constraint failed` on orphaning operations, matching Postgres/MySQL behavior.

If you see "no such table" after editing schema or lessons, check you don't have a stale Flask process on 5001 that held the old `learn.db` open before you removed it.

### Schema introspection feeds two consumers

`get_schema()` queries `sqlite_master` + `PRAGMA table_info` on every request. The result drives:

1. The schema sidebar (rendered directly in Jinja).
2. CodeMirror's `sql({ schema: … })` autocomplete (serialized via `schema | tojson`).

So when a learner runs `CREATE TABLE`, the next page render has autocomplete for the new table automatically.

### Frontend has no build step

CodeMirror 6 is loaded from **esm.sh** ESM URLs in `templates/index.html`. The imports deliberately **omit `?bundle`** — with bundling, `@codemirror/view` gets duplicated between the `codemirror` import and the separate `keymap` import, breaking the editor. Leave it alone unless you're replacing the CDN approach entirely.

`Ctrl+Space` is deliberately **not** bound for autocomplete — it collides with macOS input-source switching. `Alt/Option+Space` is the manual trigger; autocomplete also fires automatically as letters are typed.

## Workflow

All work happens directly on `master` — no feature branches, no PRs. Commit and push to `origin/master`.

---
> Source: [willbr/sql-research](https://github.com/willbr/sql-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
