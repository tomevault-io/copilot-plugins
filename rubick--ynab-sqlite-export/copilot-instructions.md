## ynab-sqlite-export

> This repo is a single-file tool: `ynab_to_sqlite.py` dumps a YNAB account

# Agent guide

This repo is a single-file tool: `ynab_to_sqlite.py` dumps a YNAB account
into `ynab.db` (SQLite). Stdlib-only Python 3, no build step, no tests to
run beyond executing the script itself.

## Working on the tool

- Keep it dependency-free and generic: no hardcoded budget IDs, category
  names, currencies, or personal assumptions — anyone's YNAB token must work.
- Incremental sync relies on YNAB's `server_knowledge` delta cursors stored
  in the `sync_state` table. If you change the schema, bump users via
  `--full` (which drops and recreates all tables) and say so in the README.
- Never commit `.env` or `*.db` (both gitignored) — they contain the
  maintainer's real token and real financial data.

## Querying a ynab.db produced by this tool

Full guidance (schema, pitfalls, ready-made SQL) lives in
[`.claude/skills/ynab-db/SKILL.md`](.claude/skills/ynab-db/SKILL.md) —
read it before writing SQL. The two mistakes that silently corrupt every
analysis: forgetting `deleted = 0` (deletes are soft) and counting
transfers (`transfer_account_id IS NOT NULL`) as income/spending.

---
> Source: [RuBiCK/ynab-sqlite-export](https://github.com/RuBiCK/ynab-sqlite-export) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
