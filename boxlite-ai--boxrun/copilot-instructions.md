## boxrun

> BoxRun is a local VM execution platform built on BoxLite. This is the Rust rewrite providing a single binary (server + CLI), a Rust SDK crate, and full API compatibility with the Python version.

# BoxRun (Rust)

## Project Overview
BoxRun is a local VM execution platform built on BoxLite. This is the Rust rewrite providing a single binary (server + CLI), a Rust SDK crate, and full API compatibility with the Python version.

## Key Architecture
- **Server**: axum + tokio on TCP or Unix socket
- **Store**: rusqlite + spawn_blocking for async SQLite (`~/.boxrun/boxrun.db`)
- **BoxManager**: Wraps BoxLite Rust SDK, tracks resource usage, manages box/exec lifecycle
- **EventBus**: In-memory pub/sub via tokio mpsc channels for SSE streaming
- **CLI**: clap derive — same binary as server (`boxrun serve` / `boxrun create` / etc.)
- **SDK**: `boxrun-sdk` crate — reqwest async client
- **Web UI**: Single-page dashboard at `/ui` — embedded via `include_str!`

## Workspace Structure
```
crates/
├── boxrun-types/     # Shared types (errors, models, config)
├── boxrun-server/    # Server + CLI binary
└── boxrun-sdk/       # Rust SDK crate
```

## Common Commands
- **Build**: `cargo build`
- **Run tests**: `cargo test --workspace`
- **Check**: `cargo check --workspace`
- **Clippy**: `cargo clippy --workspace`
- **Format**: `cargo fmt --all`
- **Run server**: `cargo run -- serve`
- **Run CLI**: `cargo run -- create ubuntu`

## API Compatibility
The Rust server implements the exact same REST API as the Python version:
- Same endpoint paths (including `POST /v1/boxes/{id}:stop`)
- Same JSON request/response fields and types
- Same 11 error codes and HTTP status code mapping
- Same SSE event format
- Python SDK connects without modifications

## BoxLite Integration
- BoxLite Rust SDK used directly (no Python wrapper needed)
- `BoxOptions`: `cpus`, `memory_mib`, `disk_size_gb`, `working_dir`, `env` (Vec<(String, String)>), `volumes`, `auto_remove=false`
- Per-exec workdir wrapped via `/bin/sh -c "cd DIR && exec CMD"` (matching Python behavior)
- Streams are take-once; stdout/stderr read concurrently via `tokio::join!`

## Bug Patterns
- SQLite: WAL mode + foreign_keys=ON via PRAGMA
- SSE race: Subscribe BEFORE DB read, dedup with seen_seqs, re-read on finish
- CancelledError: Finalize DB state in task abort handler
- WebSocket: Kill execution on disconnect
- File copy: Use `/root/` not `/tmp/` (tmpfs silently fails)

## Engineering Standards
- Run `cargo test --workspace` before declaring changes complete
- Run `cargo clippy --workspace` — no warnings allowed
- Keep changes minimal and focused
- Read existing code before modifying

---
> Source: [boxlite-ai/boxrun](https://github.com/boxlite-ai/boxrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
