## iknow-tools

> This repository contains a Go CLI for collecting iKnow! learning statistics in SQLite, comparing date ranges, and generating progress plots.

# Repository Guidelines

## Overview

This repository contains a Go CLI for collecting iKnow! learning statistics in SQLite, comparing date ranges, and generating progress plots.

## Project Structure

- `main.go`: CLI entry point and command registration.
- `internal/`: command implementations, database access, models, flags, and tests.
- `test/data/`: HTML fixtures used by tests.
- `cron.sh`: scheduled collection and report generation.
- `iknow.sqlite3`, `plot.png`, `iknow-tools`: local/generated runtime artifacts; do not modify them unless the task explicitly requires it.

## Development Commands

- Run all tests: `go test ./...`
- Format Go code: `gofmt -w <files>`
- Build the CLI: `go build -o iknow-tools .`
- Run locally: `go run . --help`

## Conventions

- Follow standard Go style and keep all changed Go files formatted with `gofmt`.
- Keep CLI wiring in `main.go`; put command behavior and reusable logic in `internal/`.
- Wrap errors with useful context using `%w`.
- Add or update focused tests for behavior changes. Prefer in-memory SQLite and `httptest` over external services.
- Preserve existing command names, flags, output formats, database schema, and generated filenames unless a change explicitly requires otherwise.

## Verification

Before finishing a code change, run `go test ./...`. Also run `go build .` when command wiring, dependencies, or build behavior changes.

## Safety

- Do not commit credentials, personal iKnow! data, or machine-specific paths.
- Avoid tests that contact `iknow.jp`; use the fixture in `test/data/` or a local test server.
- Treat the checked-in SQLite database and generated binary/image as user data or artifacts; do not overwrite them during routine verification.

---
> Source: [blue-jam/iknow-tools](https://github.com/blue-jam/iknow-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
