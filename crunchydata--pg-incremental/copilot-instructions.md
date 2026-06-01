## pg-incremental

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Install

```bash
# Ensure pg_config is in PATH (adjust version as needed)
export PATH=/usr/pgsql-17/bin:$PATH

make
sudo PATH=$PATH make install
```

## Testing

```bash
# Run all regression tests
make installcheck

# Run a single test (e.g., just the sequence tests)
make installcheck REGRESS=sequence
make installcheck REGRESS=time_interval
```

Test SQL scripts live in `sql/`, expected outputs in `expected/`, and actual outputs (on failure) in `results/`. Diffs appear in `regression.diffs`.

## Architecture Overview

**pg_incremental** is a PostgreSQL extension (C + SQL) for exactly-once incremental batch processing. It tracks progress within the same transaction as the user's command, ensuring no data is processed twice or skipped.

### Three Pipeline Types

All three share a common `incremental.pipelines` table and are dispatched through `pipeline.c`:

1. **Sequence pipelines** (`src/sequence.c`) — processes new rows by safe sequence ranges. Calls user command with `($1 min_seq, $2 max_seq)`. Uses a "safe horizon" mechanism that waits for concurrent write transactions to commit before advancing.

2. **Time interval pipelines** (`src/time_interval.c`) — processes completed time windows (e.g., `1 day`). Calls user command with `($1 start_time, $2 end_time)`. Supports a configurable `min_delay` before a window is considered stable.

3. **File list pipelines** (`src/file_list.c`) — processes files from cloud/local storage one-at-a-time or in batches. Tracks processed files in `incremental.processed_files`. Default list function is `lake_file.list` (falls back to `crunchy_lake.list_files`).

### Key Source Files

| File | Role |
|------|------|
| `src/pipeline.c` | Pipeline CRUD, `execute_pipeline()` dispatch, event triggers for DROP |
| `src/sequence.c` | Safe sequence range detection and execution |
| `src/time_interval.c` | Time window iteration and execution |
| `src/file_list.c` | File enumeration, deduplication, and execution |
| `src/cron.c` | pg_cron job creation/deletion (optional dependency since v1.3) |
| `src/query.c` | Shared query execution helpers |
| `src/init.c` | Extension entry point, GUC registration |

### SQL Schema

The `incremental` schema contains:
- `pipelines` — master table for all pipeline definitions
- `sequence_pipelines`, `time_interval_pipelines`, `file_list_pipelines` — type-specific state
- `processed_files` — deduplication log for file list pipelines

SQL schema is defined in `pg_incremental--1.0.sql`; incremental migrations are in `pg_incremental--1.X--1.Y.sql` files.

### Scheduling

Pipelines are scheduled via pg_cron with job names prefixed `pipeline:`. Since v1.3, pg_cron is a soft dependency — pipelines can be created and executed manually without it.

### Supported PostgreSQL Versions

PostgreSQL 14–17. Uses PGXS for the build system.

---
> Source: [CrunchyData/pg_incremental](https://github.com/CrunchyData/pg_incremental) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
