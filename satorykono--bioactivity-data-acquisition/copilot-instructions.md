## bioactivity-data-acquisition

> USE WHEN implementing pipelines; follow standard contract stages


Pipelines contract stages

> Scope:
> - USE WHEN implementing pipelines; follow standard contract stages
> - Use when editing files matching: `src/bioetl/pipelines/**/*.py`, `docs/etl_contract/**/*.md`
# MANDATORY
- Implement the standard ETL sequence: `extract → transform → validate → export`.

# GOOD
Each pipeline class provides these stage methods; validation precedes export.

# REFERENCE
See ../../docs/styleguide/08-etl-architecture.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SatoryKono) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
