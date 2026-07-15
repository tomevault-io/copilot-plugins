## valkey-timeseries

> - Short, focused instructions to help an AI model become productive in this codebase quickly.

# AGENTS: Guidance for AI coding agents

Purpose

- Short, focused instructions to help an AI model become productive in this codebase quickly.

Quick start (commands you can run)

- Build + checks:
  `cargo fmt --check && cargo clippy --profile release --all-targets -- -D clippy::all && RUSTFLAGS="-D warnings" cargo build --all --all-targets --release`
- Local dev script (recommended):
    - `SERVER_VERSION=unstable ./build.sh`  # builds module, builds valkey-server, runs unit & integration tests
    - To run ASAN integration pass: `ASAN_BUILD=true SERVER_VERSION=unstable ./build.sh`
    - Run a subset of Python integration tests: `TEST_PATTERN="test_ts_add" SERVER_VERSION=unstable ./build.sh`

Key ENV and behavior (from `./build.sh`)

- `SERVER_VERSION` (required): controls which valkey-server is cloned/built and stored at
  `tests/build/binaries/$SERVER_VERSION/valkey-server`. Defaults to `unstable` if not set, which tracks the latest main or branch.
- `ASAN_BUILD`: when set runs tests with LeakSanitizer checks and fails on leaks.
- `TEST_PATTERN`: passed to pytest `-k` to select tests.
- `MODULE_PATH` exported after build: `target/release/libvalkey_timeseries{.so,.dylib}` depending on OS.

Setup & Environment Notes

- Rust version: The project requires a minimum Rust version of `1.92`.
- Python tests: Integration tests use Python. Dependencies are in `requirements.txt` (or via `uv sync`). The `build.sh`
  script handles this, but if running `pytest` manually, ensure packages are installed.
- Running manually: To manually start a server with the module loaded, run
  `valkey-server --loadmodule ./target/release/libvalkey_timeseries.so` (requires building the module first).

High-level architecture (big picture)

- This is a Valkey module (Rust crate) exposing TS.* commands to the Valkey server via the `valkey_module!` macro (
  `src/lib.rs`).
- Command implementations live in `src/commands/*` and are registered in `src/lib.rs` with a one-to-one mapping to
  Valkey commands. Example:
    - `["TS.ADD", commands::ts_add_cmd, "write deny-oom", 1, 1, 1, "write timeseries"]`
- Time-series core lives under `src/series` (storage, encoding, background tasks, indexes). Index/init helpers:
  `init_croaring_allocator()` and `init_background_tasks()` are invoked from `src/lib.rs`.
- Cross-node fanout / clustering patterns: `src/fanout` and `src/commands/*_fanout_command.rs` use protobuf (
  `src/commands/fanout.*.proto`) and explicit fanout registration (`register_fanout_operations`) to implement
  cluster-wide queries.

Project-specific conventions and patterns

- All Valkey commands are declared in the `valkey_module!` macro in `src/lib.rs`; change there to add/remove commands.
- Command files follow `ts_<command>.rs` naming and export `ts_<command>_cmd` functions (see `src/commands/mod.rs`).
- Fanout pattern: synchronous local implementation + `*_fanout_command.rs` files which marshal/unmarshal protobuf
  messages for cluster aggregation.

- Allocator in tests: tests compile with `enable-system-alloc` feature in unit runs (see `build.sh`), and the crate
  switches allocators under `#[cfg(test)]`.

Testing & debugging notes

- Unit tests: `cargo test --features enable-system-alloc`.
- Doc tests: `cargo test --doc --features enable-system-alloc`.
- Integration tests: Python pytest under `tests/` and rely on a built `valkey-server` and `tests/valkeytestframework`
  helper files (populated by `./build.sh`).
- To reproduce integration runs locally: run `SERVER_VERSION=unstable ./build.sh` — this will clone/build Valkey and
  copy the server binary to `tests/build/binaries/`.
- Leak detection: when `ASAN_BUILD` is set, the build script scans pytest output for LeakSanitizer output and fails if
  leaks are detected.

Where to look first (key files & directories)

- `src/lib.rs` — module entrypoint, command registration, lifecycle (preload/init/deinit).
- `src/commands/` — implementations and command parsing utilities (`command_parser.rs`).
- `src/series/` — core storage, encodings, indexes, background tasks.
- `src/fanout/` — cluster communication primitives.
- `build.sh` — canonical developer flow for formatting, linting, building, and running tests.
- `README.md` and `docs/commands/` — human-facing command descriptions and examples.

Quick tips for code changes

- Add new commands: create `src/commands/ts_<name>.rs`, add function `ts_<name>_cmd`, then register in `valkey_module!`
  in `src/lib.rs`.
- Documentation: When adding or modifying commands, remember to update the human-facing docs in `docs/commands/` and the
  supported list in `README.md`.
- When making cluster changes, search for `*_fanout_command.rs` to copy the fanout pattern and add protobuf messages in
  `src/commands/fanout.*.proto`.

Limitations of this document

- Focused on discoverable, executable patterns. It does not cover domain rationale beyond what is visible in
  source/docs.

If you need more context, inspect:

- `build.sh`, `Cargo.toml`, `src/lib.rs`, `src/commands/*`, `src/series/*`, and `tests/`.

---
> Source: [ccollie/valkey-timeseries](https://github.com/ccollie/valkey-timeseries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
