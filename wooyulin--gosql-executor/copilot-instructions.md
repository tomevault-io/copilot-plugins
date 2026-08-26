## gosql-executor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

gosql-executor — a lightweight CLI SQL query tool supporting MySQL, Oracle, and PostgreSQL. Executes interactive SQL queries and exports results to CSV or JSON. Written in Go, built as a static binary with `CGO_ENABLED=0` (pure Go Oracle driver, no OCI dependency).

## Build & Run

```bash
# Build
go build -o sql-executor.exe ./cmd        # Windows
go build -o sql-executor ./cmd            # Linux/macOS

# Cross-compile for Linux
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o sql-executor ./cmd

# Run (enters interactive setup if config.yaml missing)
./sql-executor
./sql-executor --config=/path/to/config.yaml
```

No unit tests exist. Integration testing is done via `test/test.sh` which SSHes into remote servers using `plink`/`pscp`.

## Architecture

```
cmd/main.go              → Entry point: parse flags, load config, connect DB, launch REPL
config/                  → Config structs (viper/yaml), interactive first-run setup (liner)
internal/database/       → Database interface + factory, MySQL/Oracle/PostgreSQL implementations
internal/executor/       → SQL REPL loop (liner), multi-line input (; terminated), query dispatch
internal/output/         → Console table (tabwriter, 100-row limit), CSV/JSON file writers (1GB cap)
pkg/logger/              → Simple Info/Error/Fatal logger wrapper
```

### Key patterns

- **Database factory**: `database.NewDatabase(cfg)` returns the right implementation based on `cfg.Type` (`mysql`/`oracle`/`pgsql`). All drivers embed `BaseDatabase`.
- **DSN templating**: Config DSN uses `{username}` and `{password}` placeholders, substituted at runtime with URL-encoding for Oracle.
- **Oracle semicolon handling**: `executor.processQuery()` strips trailing `;` for Oracle (driver rejects them), re-adds for MySQL/PostgreSQL.
- **Config**: YAML via viper. Struct tags use both `yaml` and `mapstructure`. Interactive setup uses `liner` for terminal prompts with masked password input.

## CI/CD

GitHub Actions (`.github/workflows/build.yml`): triggers on `v*` tag push or manual dispatch. Builds linux/amd64, windows/amd64, darwin/amd64. Packages binary + `config.yaml.example` into zip, uploads to GitHub Releases.

## Module

Module name is `sql-executor` (local, not the GitHub path). Key deps: `go-sql-driver/mysql`, `lib/pq`, `sijms/go-ora/v2`, `spf13/viper`, `peterh/liner`.

---
> Source: [Wooyulin/gosql-executor](https://github.com/Wooyulin/gosql-executor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
