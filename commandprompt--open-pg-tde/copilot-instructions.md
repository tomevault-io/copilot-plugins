## open-pg-tde

> `open_pg_tde` is an open fork of Percona's `pg_tde`, maintained by Command Prompt, Inc.

# open_pg_tde project instructions

`open_pg_tde` is an open fork of Percona's `pg_tde`, maintained by Command Prompt, Inc.
It provides Transparent Data Encryption for upstream PostgreSQL 16 and later.

## Coding standards (required)

All code in this project MUST satisfy the PostgreSQL coding conventions:
https://www.postgresql.org/docs/current/source.html

This applies to every C change, in both the extension and the core patch under
`patches/postgresql/`. Follow PostgreSQL's formatting, brace, naming,
error-reporting (`ereport`/`errcode`), and memory-management (palloc/memory
context) conventions, and run `pgindent` where applicable. Match the style of
the surrounding PostgreSQL code. This is a firm requirement for every
contribution and every review.

## How the project is built

- Runs on upstream PostgreSQL 16/17/18 (16 is the floor; the extension has no
  support below 16). It is NOT tied to a vendor server fork.
- The core storage-manager and WAL extensibility is applied as a gated patch to
  a stock PostgreSQL source tree: `patches/postgresql/<major>/`, applied with
  `patches/postgresql/apply.sh`.
- The patch is gated behind `USE_TDE_HOOKS` (`--enable-tde-hooks` /
  `-Dtde_hooks`). With the flag off, the patched tree builds as unmodified
  PostgreSQL. Verify both states with `patches/postgresql/verify-gate.sh`.
- Docs are Material for MkDocs under `documentation/`.

## Documentation style

No em-dashes. Avoid unnecessary adjectives. Keep prose professional and factual.

---
> Source: [commandprompt/open_pg_tde](https://github.com/commandprompt/open_pg_tde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
