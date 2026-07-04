## print-farm-manager

> - On every new session, read `docs/README.md` first to understand the project structure and existing documentation.

# CLAUDE.md — Print Farm Manager

## Documentation Rules

- On every new session, read `docs/README.md` first to understand the project structure and existing documentation.
- After implementing any feature, bugfix, or significant change:
  - Update the relevant component doc in `docs/`.
  - Add a dated entry to `docs/CHANGELOG.md` with: what changed, why, and any new dependencies or configuration.
  - Update `docs/README.md` if new docs were created or the structure changed.
- Follow the documentation format and style already established in existing docs.
- When creating new components or modules, create a matching doc file in `docs/` before writing code.

## Project Conventions

- All DB queries use `better-sqlite3` synchronous API — no `async/await` for database operations.
- All timestamps are Unix epoch milliseconds (`Date.now()`).
- Booleans in SQLite are `INTEGER` (`0`/`1`).
- Route modules export a factory function `(db) => router` and are mounted in `server/index.js`.
- Partial updates use `COALESCE(?, column)` — omitting a field leaves the existing value intact.
- Model uniqueness on `(part_id, printer_model)` is enforced at the application layer, not as a DB constraint.
- Do not add a DB migration system — Phase 1 uses `CREATE TABLE IF NOT EXISTS` only.

## Phase Scope

Currently in **Phase 1**. Do not implement Phase 2+ features unless explicitly instructed. Phase 2 scope (job scheduling, dispatch, G-code upload, Part state machine) is defined in `ARCHITECTURE.md` Section 11.7.

---
> Source: [joeltelling/print-farm-manager](https://github.com/joeltelling/print-farm-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
