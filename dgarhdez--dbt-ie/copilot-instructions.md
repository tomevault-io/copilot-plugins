## dbt-ie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **teaching repository** for an Analytics Engineering with dbt course (15 sessions), targeting dbt Analytics Engineering Certification preparation. It uses **DuckDB** as the database backend with an ecommerce dataset (14 source tables as Parquet files in `data/`).

## Common Commands

```bash
# Environment setup (dependencies in pyproject.toml, Python pinned to >=3.10,<3.13)
uv sync                     # Creates .venv/, resolves deps, writes uv.lock
source .venv/bin/activate   # macOS/Linux (Windows: .\.venv\Scripts\Activate.ps1)
# Or skip activation entirely and prefix commands with `uv run`, e.g. `uv run dbt debug`
dbt deps                    # Install dbt packages (from packages.yml)

# Database setup
python create_db.py         # Load Parquet files into DuckDB (creates my_database.duckdb)

# dbt commands
dbt debug                   # Verify connection
dbt run                     # Build all models
dbt run -s staging          # Build only staging layer
dbt run -s +mart_orders     # Build mart_orders and all upstream dependencies
dbt run -s model_name       # Build a single model
dbt test                    # Run all tests
dbt test -s model_name      # Test a single model
dbt seed                    # Load seed files (seeds/segments.csv)
dbt build                   # Run + test + seed + snapshot in DAG order

# SQL linting
sqlfluff lint models/       # Lint SQL files
sqlfluff fix models/        # Auto-fix SQL files

# Slide decks — convert a Marp markdown file to PDF (images in ../img/ are referenced with relative paths)
npx --yes @marp-team/marp-cli class_material/<file>.md --pdf --allow-local-files
# Example: npx --yes @marp-team/marp-cli class_material/01_intro_ae_dbt_structure.md --pdf --allow-local-files

# Diagrams — Mermaid sources live in img/diagrams/*.mmd and render to sibling *.svg files.
./render_diagrams.sh              # re-render every .mmd in img/diagrams/
./render_diagrams.sh layers       # re-render one (matches basename without .mmd)
# Reference in slides as:  ![center w:1100](../img/diagrams/<name>.svg)
# Always re-render diagrams before rebuilding PDFs if the .mmd source changed.
```

## Architecture

**Three-layer model DAG**: Sources → Staging → Intermediate → Marts

- **Sources** (`models/sources.yml`): 14 raw tables in DuckDB `main` schema, loaded from Parquet files via `create_db.py`
- **Staging** (`models/staging/`): 13 view models (`stg_*`) — thin wrappers over sources with `select * from {{ source('raw', ...) }}`; `stg_customers` is materialized as a table
- **Intermediate** (`models/intermediate/`): 3 models (`int_*`) — joins, window functions, aggregations that combine staging models
- **Marts** (`models/marts/`): 7 models — `dim_*` (dimensions), `mart_*` (facts/aggregates), plus one Python model (`customers_enriched_python.py` using Polars)

Each layer has a `docs/` subdirectory with YAML files containing column descriptions and tests (unique, not_null, relationships).

## Key Conventions

- **SQL style**: lowercase keywords, identifiers, functions, literals (enforced by `.sqlfluff`)
- **Naming**: `stg_[table]`, `int_[descriptive]`, `dim_[entity]`, `mart_[metric]`
- **Materialization**: staging = views, marts = tables (configured in `dbt_project.yml`)
- **dbt profile**: located at `profiles.yml` in the project root. dbt 1.5+ finds it automatically when commands are run from the project directory, overriding any global `~/.dbt/profiles.yml`. Adapter is `duckdb`, database file is `my_database.duckdb`.
- **Packages**: dbt_utils, codegen, dbt_expectations (see `packages.yml`)

## Course Materials

`class_material/` contains 15 session slide decks in Marp markdown format. `documentation/syllabus_2.md` (current syllabus, aligned with dbt Fundamentals cert) and `documentation/certification.md` provide the course outline and certification prep details. `documentation/publishing_strategy.md` (gitignored, instructor-only) describes how the repo is published to students. `notebooks/` has Jupyter notebooks for DuckDB exploration and Python model tutorials.

---
> Source: [dgarhdez/dbt-ie](https://github.com/dgarhdez/dbt-ie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
