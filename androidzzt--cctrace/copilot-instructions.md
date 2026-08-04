## cctrace

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

- Run all tests: `go test ./...`
- Run trace model tests only: `go test ./internal/trace`
- Run a single test: `go test ./internal/app -run TestParseArgsForWrappedCommand`
- Run the CLI in development: `go run ./cmd/cctrace Codex -- <command>`
- Run a smoke trace: `go run ./cmd/cctrace Codex -- sh -c 'printf cctrace-smoke'`
- Replay a saved session: `go run ./cmd/cctrace view <session-id>`

## Architecture

`cctrace` is a Go CLI and local web UI for profiling AI coding agent sessions. The CLI wraps Codex or Codex commands, records process and trace events, stores sessions as JSONL, and serves a local timeline UI.

Core packages:

- `internal/trace` defines the shared `Event` and `Session` model.
- `internal/store` persists sessions and events under a session directory.
- `internal/collectors` converts process, transcript, and ccglass records into trace events.
- `internal/correlate` links events heuristically and marks confidence.
- `internal/server` serves the local web UI and event JSON API.
- `internal/app` wires CLI parsing, collection, persistence, and server startup.

---
> Source: [androidZzT/cctrace](https://github.com/androidZzT/cctrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
