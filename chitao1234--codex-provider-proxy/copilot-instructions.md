## codex-provider-proxy

> This repository is a Rust workspace for a local HTTP reverse proxy that routes requests to different upstream

# AGENTS.md

This repository is a Rust workspace for a local HTTP reverse proxy that routes requests to different upstream
"providers" (base URL + API key). The special routing behavior is based on the **PID of the client process**
that opened the TCP connection (Linux `/proc`-based implementation and Windows `GetExtendedTcpTable`-based
implementation), with a local RPC server to manage PID→provider mappings at runtime.

## Project Structure

Workspace root:

- `Cargo.toml` — workspace members and shared dependency versions
- `config.example.toml` — example runtime configuration
- `README.md` — usage and behavior overview

Crates:

- `crates/pid-resolver/`
  - Purpose: resolve the PID for the *peer end* of a loopback TCP connection
  - Key files:
    - `crates/pid-resolver/src/lib.rs` — `PidResolver` trait
    - `crates/pid-resolver/src/platform/` — platform implementations (`linux.rs`, `windows.rs`)
    - `crates/pid-resolver/tests/resolves_peer_pid.rs` — integration test (OS-specific assertions)
    - `crates/pid-resolver/src/bin/pid_resolver_test_client.rs` — helper binary used by integration tests

- `crates/proxy/` (binary: `codex-provider-proxy`)
  - Purpose: reverse proxy server + RPC management server
  - Key files:
    - `crates/proxy/src/main.rs` — starts proxy listener + RPC listener
    - `crates/proxy/src/config.rs` — config parsing, validation, example config text
    - `crates/proxy/src/proxy.rs` — request forwarding, auth header rewrite, path prefix stripping, logging
    - `crates/proxy/src/rpc.rs` — local RPC endpoints to set/list/clear PID routes
    - `crates/proxy/src/log_capture.rs` — bounded body capture for logging

- `crates/rpc-types/` (library: `codex-provider-proxy-rpc-types`)
  - Purpose: serde request/response types shared between server and client
  - Key file: `crates/rpc-types/src/lib.rs`

- `crates/proxyctl/` (binary: `codex-provider-proxyctl`)
  - Purpose: CLI RPC client to manage PID routes
  - Key files:
    - `crates/proxyctl/src/main.rs` — CLI entrypoint, RPC client, and `match` subcommand
    - `crates/proxyctl/src/process_scan.rs` — local process scan helpers (Linux `/proc`; Windows Toolhelp + PEB-based cwd)
    - `crates/proxyctl/src/bin/process_scan_test_target.rs` — helper binary used by Windows scan tests
    - `crates/proxyctl/tests/process_scan_finds_child_by_cmdline_windows.rs` — Windows integration test for process scanning
    - `crates/proxyctl/tests/process_scan_finds_self.rs` — Linux/Windows integration test for process scanning

## Key Behaviors / Invariants

- **Local-only:** both the proxy listener and RPC listener are intended to be loopback-only.
- **PID selection:** provider selection is based on the *client/peer PID* (not the proxy process PID).
- **Default routing:** if no PID mapping exists, requests route to `default_provider`.
- **Authorization rewrite:** outgoing `Authorization` is replaced using the selected provider configuration.
- **Path prefix stripping:** `listen_base_path` (e.g. `"/v1"`) is stripped from the incoming URL path before
  joining onto the provider `base_url` path (e.g. `https://example.com/v2`).

## Development Commands

- Format: `cargo fmt`
- Tests: `cargo test`
- Run proxy: `cargo run -p codex-provider-proxy -- --config config.toml`
- Print example config: `cargo run -p codex-provider-proxy -- --print-example-config`
- Manage routes (auto-reads `./config.toml` if present):
  - Set: `cargo run -p codex-provider-proxyctl -- set --pid $$ --provider provider_b`
  - List: `cargo run -p codex-provider-proxyctl -- list`

## Implementation Notes (for future contributors/agents)

- Keep platform-specific PID logic behind `PidResolver` in `crates/pid-resolver` to leave room for future
  multi-platform support.
- Prefer minimal, focused changes. Update `config.example.toml` and `README.md` when behavior/config changes.
- When modifying proxy forwarding behavior, ensure hop-by-hop headers remain filtered and body streaming stays
  streaming (avoid buffering full bodies unless explicitly requested).

---
> Source: [chitao1234/codex-provider-proxy](https://github.com/chitao1234/codex-provider-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
