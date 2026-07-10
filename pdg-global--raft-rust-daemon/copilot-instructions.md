## raft-rust-daemon

> This file is for AI assistants working on the Rust port of the Raft daemon.

# Agent Notes for raft-daemon-rust

This file is for AI assistants working on the Rust port of the Raft daemon.

## Project overview

- Rust port of the npm package `@botiverse/raft-daemon` (version `0.69.0`).
- Crate name: `raft-daemon`.
- Version is defined in `Cargo.toml` and is the source of truth for releases.
- Repository: https://github.com/PDG-Global/raft-rust-daemon

## Tech stack

- Rust edition 2024, MSRV 1.85.
- Async runtime: `tokio`.
- WebSocket client: `tokio-tungstenite` with `rustls-tls-native-roots`.
- HTTP client: `reqwest` (`.no_proxy()` is set).
- Serialization: `serde`/`serde_json`.
- Tracing: `tracing`/`tracing-subscriber`.
- Testing: built-in `cargo test` plus unit tests under `tests/unit/`.

## Common commands

```bash
# Run tests
cargo test

# Check / clippy
cargo check
cargo clippy

# Debug build
cargo build

# Optimised release build (native target)
cargo build --release

# Cross-compile for a specific target
rustup target add aarch64-unknown-linux-gnu
cargo build --release --target aarch64-unknown-linux-gnu

# Full cross-compilation release script (macOS / Linux / FreeBSD)
./build-release.sh
```

## Release process

1. Ensure `Cargo.toml` version matches the intended release tag.
2. Run `./build-release.sh` to produce binaries under `dist/`.
3. Update `README.md` and `CHANGELOG.md` if needed.
4. Commit and push to `main`.
5. Create a GitHub release with the tag `v<version>` (e.g. `v0.69.0`).
6. Upload release binaries and include SHA-256 hashes in the release notes.
7. **macOS only:** upload the per-architecture `arm64` and `x86_64` binaries. Do **not** upload the `universal` macOS binary unless the codesign/timestamp issues are fully resolved.

## Runtime drivers

- The daemon advertises runtimes to the raft server via the `ready` frame.
- `builtin` is the default runtime and is backed by RustyCLI. RustyCLI **must** be installed on the host or the runtime list is empty.
- RustyCLI is discovered from `$RAFT_RUSTY_BINARY`, then `rusty`, `rustycli`, or `rusty-cli` on `PATH`.
- The `rusty` runtime is also available and uses the same RustyCLI binary under the hood; the difference is only the runtime name advertised to the server.

## Key architecture files

- `src/main.rs` - CLI dispatch and daemon entry point.
- `src/cli/{args,commands}.rs` - CLI flags and command handlers.
- `src/daemon/mod.rs` - Core daemon control (`start`, `stop`, `status`, `restart`).
- `src/daemon/agent/mod.rs` - WebSocket connection and message dispatch.
- `src/daemon/agent/process.rs` - Agent lifecycle, `AgentProcessRegistry`, RustyCLI spawn.
- `src/daemon/agent/raft_client.rs` - HTTP client and `derive_target` for reply routing.
- `src/daemon/runner.rs` - Lifecycle frame construction (`agent:session`, `agent:status`, `agent:activity`).
- `src/daemon/pidfile.rs` - PID file handling.
- `src/daemon/paths.rs` - Home directory layout (`~/.raft-daemon/`).
- `src/runtime/` - Runtime driver abstraction and the `builtin`/`rusty` drivers.
- `tests/unit/cli.rs` - Unit tests for CLI argument parsing.

## Important implementation details

- The daemon backgrounds itself by default with `setsid()` unless `--foreground` is passed.
- Home directory defaults to `~/.raft-daemon/` and can be overridden with `RAFT_DAEMON_HOME`.
- Per-agent workspace lives at `~/.raft-daemon/agents/<agent_id>/`.
- `MEMORY.md` and `notes/` are created in the agent workspace on `agent:start`.
- If a legacy npm daemon home exists at `~/.slock/agents/<agent_id>/`, `MEMORY.md` and `notes/` are migrated into the new workspace on first start.
- `MEMORY.md` is injected into every RustyCLI prompt (truncated at 8 KB), so the agent retains context across turns.
- The WebSocket path connects to `<server_url>/daemon/connect?key=<api_key>`.
- `ring` CryptoProvider is installed explicitly at startup to avoid rustls panics.
- Provider config is extracted from multiple shapes/casings (`config.provider`, `config.runtimeConfig.provider`, snake_case and camelCase fields, model-prefix inference).
- The `ready` frame must include `runtimes`, `capabilities`, `os`, `hostname`, `daemonVersion`, and `computerVersion` (via `RAFT_COMPUTER_VERSION`).
- Lifecycle frames should omit `launchId` when it is absent, matching the npm package.
- `agent:activity` frames must include `clientSeq` and `producerFactId`.

## Known issues / caveats

- The macOS universal binary produced by `build-release.sh` can fail codesign with `A timestamp was expected but was not found.`, even though the individual `arm64` and `x86_64` binaries sign successfully. Do not ship the universal macOS binary until resolved; ship the two single-arch binaries instead.
- Proxy egress is not fully honored by the HTTP client; `tokio-tungstenite` does honor `HTTPS_PROXY`.
- Each `agent:deliver` spawns a fresh `rusty` process; continuity relies on `--resume <session_id>` and the agent workspace files (especially `MEMORY.md`).
- `agent:deliver` now filters out self-echo and bot messages in public channels, and injects `MEMORY.md` context into the prompt. If the server sends different payload shapes, the debug log of the raw delivery will show them.

## Testing conventions

- Run the full suite before committing: `cargo test`.
- Also run `cargo clippy`.
- When adding agent lifecycle or frame logic, update or add tests in `tests/unit/` and the integration-style tests in the relevant `src/daemon/agent/` modules.

---
> Source: [PDG-Global/raft-rust-daemon](https://github.com/PDG-Global/raft-rust-daemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
