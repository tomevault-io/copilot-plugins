## helpline

> B2B helpdesk platform. Monorepo: 5 services + 2 shared packages. Python 3.11+, `uv`.

# Helpline

B2B helpdesk platform. Monorepo: 5 services + 2 shared packages. Python 3.11+, `uv`.

This root file is intentionally lean — it holds only what's true everywhere.
Each service and package has its own `CLAUDE.md` with local conventions;
Claude loads them automatically as it moves into that directory.

## Where things live

See `CODEBASE_MAP.md` for the full tree. Top level:

- `services/` — `api`, `auth`, `billing`, `notifications`, `search`
- `packages/` — `core` (domain models + errors), `db` (connection + repos)

## Critical gotchas (repo-wide)

- **`packages/core` and `packages/db` are imported by every service.** A change
  there ripples everywhere — run the full suite, not a scoped one, for those.
- **Run tests scoped to what you changed.** `uv run pytest services/auth` after
  an auth change. The full suite is only for `core`/`db` changes. See each
  service's `CLAUDE.md` for its exact command.
- **Imports are flat.** `core.*`, `db.*`, and each service name (`api`, `auth`,
  …) are importable directly — `packages/` and `services/` are on the path.
- **The DB connection is a process-wide in-memory stub.** `get_connection()`
  returns one shared instance. Tests that mutate tables must clean up after
  themselves or state leaks across tests.
- **Navigate by symbol, not by grep.** pyright runs as the language server —
  to find where something is defined or used, prefer LSP go-to-definition and
  find-references over text search. In a codebase this size grep returns false
  matches (comments, substrings, unrelated names); the LSP is exact. Setup and
  verification: `docs/lsp-setup.md`.

## Commands

```bash
uv sync --extra dev      # install
uv run pytest -q         # full suite
npx pyright              # type-check
```

---
> Source: [coleam00/helpline](https://github.com/coleam00/helpline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
