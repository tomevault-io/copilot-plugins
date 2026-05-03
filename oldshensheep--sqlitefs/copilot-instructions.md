## sqlitefs

> This project is a FUSE filesystem built on SQLite. It supports snapshots, compression, and

# AGENTS.md

## Project

This project is a FUSE filesystem built on SQLite. It supports snapshots, compression, and
deduplication.

## Rules

- Keep modules small and focused. Avoid mixing responsibilities.hen a change affects compatibility, make the compatibility impact explicit and do not preserve
  old behavior unless requested.
- Prefer clear structure over large, catch-all files.
- Prefer compile-time safety over runtime failure where practical.
- Return clear errors and use structured logging. Do not fail silently.
- Add or update tests if necessary for every behavior change.
- Prefer readable Rust: simple control flow, minimal magic, and raw strings like `r#"..."#` when
  they improve clarity.
- Do not modify the database schema without approval.
- When a change affects compatibility, make the compatibility impact explicit and do not implement
  migration logic.
- After changes are complete and all tests pass, ask for approval to create a git commit.
- Write portable code when possible.

## Structure

- Keep FUSE integration, storage logic, compression, deduplication, and snapshot logic in separate
  modules.
- Keep SQLite access isolated behind well-defined interfaces.
- Avoid putting core logic directly in CLI or mount entrypoints.
- Put shared types and error definitions in dedicated modules.
- Keep tests close to the behavior they validate when practical.

---
> Source: [oldshensheep/sqlitefs](https://github.com/oldshensheep/sqlitefs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
