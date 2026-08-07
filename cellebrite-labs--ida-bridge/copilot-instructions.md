## ida-bridge

> Two runtime contexts share one package:

# ida-bridge

## Runtime model

Two runtime contexts share one package:
- Host: CLI (`cli*.py`), server (`server.py`), supervisor, agent client. Runs in the user's Python.
- IDA: exec environment (`ida_runtime.py`), idb helpers (`idb.py`), SQL layer (`sql/`), plugin. Runs in UI IDA and headless `idalib` environments.

- If you changed host-side code, verify it in host tests or host execution.
- If you changed IDA-side code, verify it in UI IDA or `idalib`.
- Don't treat host-side passing tests as proof that an IDA-side change works.
- Don't solve an IDA-side problem by adding host-side machinery unless the problem is actually on the host side.

## Source of truth

SQL design contract: `docs/sql-interface-design.md`. It owns the SQL invariants,
schema design rules, scalar naming, the mental model (rowid, u64 boundary), and
the recipe for adding write support. Read it before changing SQL tables, columns,
scalar functions, pushdown, or write support, and update it when any of that
changes. Keep it specific to this codebase -- no generic SQL or SQLite tutoring.

For IDA API semantics, constants, and struct layouts, use the [IDA SDK](https://github.com/HexRaysSA/ida-sdk) (`src/include/`, `src/plugins/idapython/`).
Local checkout: `~/sources/ida/ida-sdk`.

The SQL-as-IDA-interface approach originates with Elias Bachaalany (credited in `README.md`).

Study [idasql](https://github.com/allthingsida/idasql) and [libxsql](https://github.com/0xeb/libxsql) to understand and compare approaches, not as a spec to follow:
- `~/sources/ida/idasql`
- `~/sources/ida/libxsql`

If SDK and idasql diverge on IDA API behavior, prefer SDK.

## Development note

Use `uv run <command>` to run tools in this repository (e.g., `uv run pytest`).

Three reload scopes:
- IDA-side code: restart the IDA or `idalib` instance.
- Bridge server code: restart the bridge.
- Client/CLI code: no restart; next invocation picks up changes.

## Testing

Read `docs/testing.md` before adding or changing tests: tier scope, the fixture model, and adding fixtures.

Do not mock IDA modules (`sys.modules["ida_*"]`) in unit tests. If behavior requires IDA to verify, write an e2e test.

## Update checklist

When changing behavior, CLI surface, or SQL tables/functions:
- update `skills/ida-bridge/SKILL.md` and the reference docs under `skills/ida-bridge/references/`
- update `README.md` and relevant docs under `docs/`
- update dependent skills in other repositories that reference ida-bridge

---
> Source: [cellebrite-labs/ida-bridge](https://github.com/cellebrite-labs/ida-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
