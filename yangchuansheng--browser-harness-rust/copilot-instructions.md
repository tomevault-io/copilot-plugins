## browser-harness-rust

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Browser Harness Rust Upstream Sync**

This project is a Rust-native reimplementation of `browser-use/browser-harness`. The current work is a brownfield sync effort: replicate all applicable upstream behavior from `browser-use/browser-harness` commits after April 21, 2026 into the Rust runtime without forcing the Python repository layout onto this codebase.

The audience is agents and developers who need the Browser Harness thesis as a durable Rust runtime: typed CLIs, daemon/control-plane crates, reusable domain knowledge, and optional WASM guest workflows.

**Core Value:** The Rust implementation must preserve behavior parity with upstream Browser Harness updates while remaining idiomatic, typed, and maintainable in the existing Rust architecture.

### Constraints

- **Architecture**: Preserve the Rust workspace and crate boundaries — the migration must adapt upstream behavior into existing Rust crates and binaries.
- **Traceability**: Maintain a migration audit with upstream commit references and applicability decisions.
- **Safety**: Do not expose secrets from upstream docs or local environment; scan generated docs and changed files before commit.
- **Verification**: Run `cargo fmt --check`, `cargo test --workspace`, and targeted CLI smoke checks where possible.
- **Network dependency**: Upstream commit analysis depends on the fetched `upstream/main` remote.
- **Recording**: For recording or video tasks, follow `SKILL.md` and
  `interaction-skills/make-video.md`. A natural request to show, record, or demo
  the work opts in; significant work alone does not. Keep the exact path returned
  by `start_recording()` and never reenact a finished task. The Rust recording
  engine is not yet ported.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Rust 2021 edition - active runtime, CLI, daemon, protocol, CDP client, WASM host, and smoke runner live in `rust/Cargo.toml`, `rust/bins/*/src/main.rs`, and `rust/crates/*/src/lib.rs`.
- Rust package version `0.1.0` - inherited from `[workspace.package]` in `rust/Cargo.toml` by workspace members such as `rust/crates/bh-protocol/Cargo.toml` and `rust/bins/browser-harness-cli/Cargo.toml`.
- WebAssembly Text (WAT) - minimal guest samples are stored in `rust/guests/navigate_and_read.wat`, `rust/guests/persistent_counter.wat`, and `rust/guests/persistent_browser_state.wat`.
- Rust-to-WASM guest crates - packaged guest workflows live under `rust/guests/rust-*/Cargo.toml` and compile as `cdylib` for `wasm32-unknown-unknown`.
- Markdown - operator docs and knowledge packages are first-class artifacts in `SKILL.md`, `install.md`, `docs/*.md`, `domains/<site>/skill.md`, and `interaction-skills/*.md`.
- Shell/YAML - repository maintenance and CI are defined in `scripts/scan_sensitive.sh` and `.github/workflows/ci.yml`.
- Python - not an active runtime layer; only optional subprocess wrapper guidance exists in `docs/python-integration.md`.
## Runtime
- Rust stable toolchain - CI installs stable via `.github/workflows/ci.yml`; no `rust-toolchain` file is detected in the repo.
- Native CLI runtime - `browser-harness` routes commands to `bhctl` or `bhrun` in `rust/bins/browser-harness-cli/src/main.rs`.
- Long-lived daemon runtime - `bhd` owns the browser websocket and session state through `rust/bins/bhd/src/main.rs` and `rust/crates/bh-daemon/src/lib.rs`.
- Unix socket IPC - daemon files are `/tmp/bu-<name>.sock`, `/tmp/bu-<name>.pid`, and `/tmp/bu-<name>.log` from `rust/crates/bh-discovery/src/lib.rs`; `bhrun` connects through `std::os::unix::net::UnixStream` in `rust/bins/bhrun/src/main.rs`.
- Browser runtime - local Chrome, Chromium, and Microsoft Edge profiles are discovered via `DevToolsActivePort` in `rust/crates/bh-discovery/src/lib.rs`; remote Browser Use sessions are supported through `rust/crates/bh-remote/src/lib.rs`.
- WASM guest runtime - `bhrun run-guest` and `bhrun serve-guest` execute `.wat`/`.wasm` guests through Wasmtime in `rust/bins/bhrun/src/main.rs` and `rust/crates/bh-wasm-host/src/lib.rs`.
- Cargo - workspace root is `rust/Cargo.toml`; all runtime crates and binaries are Cargo workspace members.
- Lockfile: present at `rust/Cargo.lock`; use it as the dependency source of truth for resolved crate versions.
## Frameworks
- Tokio `1.52.1` - async runtime and Unix listener support for daemon/control paths in `rust/Cargo.toml`, `rust/crates/bh-daemon/src/lib.rs`, and `rust/bins/bhctl/src/main.rs`.
- Wasmtime `30.0.2` - WASM guest execution engine used by `rust/bins/bhrun/src/main.rs` and declared in `rust/Cargo.toml`.
- tokio-tungstenite `0.24.0` - CDP websocket transport in `rust/crates/bh-cdp/src/lib.rs`.
- reqwest `0.12.28` with `json` and `rustls-tls` - Browser Use API client and runner `http-get` implementation in `rust/crates/bh-remote/src/lib.rs` and `rust/bins/bhrun/src/main.rs`.
- serde `1.0.228` and serde_json `1.0.149` - JSON protocol serialization across `rust/crates/bh-protocol/src/lib.rs`, `rust/crates/bh-wasm-host/src/lib.rs`, and all CLI payload handling.
- Rust built-in test harness - run all unit tests with `cargo test --workspace --manifest-path rust/Cargo.toml` as documented in `docs/development.md` and `.github/workflows/ci.yml`.
- `bhsmoke` smoke runner - browser/live-site verification scenarios live in `rust/bins/bhsmoke/src/main.rs` and are documented in `rust/README.md`.
- GitHub Actions - CI executes secret scanning, Rust tests, install verification, and CLI entry-point checks in `.github/workflows/ci.yml`.
- Cargo build/install - `browser-harness install` builds `browser-harness`, `bhctl`, `bhrun`, and `bhd` from `rust/bins/browser-harness-cli/src/main.rs` into `$CARGO_HOME/bin` or a supplied install root.
- rustup WASM target - guest builds require `wasm32-unknown-unknown` as documented in `rust/README.md`, `docs/wasm-guests.md`, and `docs/future-wasm.md`.
- Formatting - use `cargo fmt --all --manifest-path rust/Cargo.toml` from `docs/development.md`.
- Secret hygiene - use `./scripts/scan_sensitive.sh`, invoked by `.github/workflows/ci.yml`.
## Key Dependencies
- `tokio` `1.52.1` - async networking, daemon listener, timers, and CLI admin flows; declared in `rust/Cargo.toml`.
- `serde` `1.0.228` / `serde_json` `1.0.149` - all daemon, runner, Browser Use API, and guest-host messages are JSON; declared in `rust/Cargo.toml`.
- `tokio-tungstenite` `0.24.0` and `futures-util` `0.3.32` - CDP websocket send/receive loops in `rust/crates/bh-cdp/src/lib.rs`.
- `reqwest` `0.12.28` - Browser Use API calls in `rust/crates/bh-remote/src/lib.rs` and generic HTTP GET operations in `rust/bins/bhrun/src/main.rs`.
- `wasmtime` `30.0.2` - guest module loading and host imports in `rust/bins/bhrun/src/main.rs`.
- `libc` `0.2.185` - Unix process/socket support for daemon lifecycle paths in `rust/crates/bh-daemon/Cargo.toml`.
- `base64` `0.22.1` - smoke verification utilities in `rust/bins/bhsmoke/Cargo.toml`.
- `rustls` / `webpki-roots` - TLS stack pulled by `reqwest` and `tokio-tungstenite`, resolved in `rust/Cargo.lock`.
- Internal workspace crates - use `bh-protocol`, `bh-discovery`, `bh-cdp`, `bh-daemon`, `bh-remote`, `bh-wasm-host`, and `bh-guest-sdk` from `rust/crates/*/Cargo.toml` instead of adding duplicate protocol or browser layers.
## Configuration
- `BU_NAME` - daemon namespace for socket, pid, and log paths; used in `rust/bins/bhd/src/main.rs`, `rust/bins/bhctl/src/main.rs`, and `rust/crates/bh-discovery/src/lib.rs`.
- `BU_CDP_WS` - explicit CDP websocket override for remote browsers or pinned local attach; read by `rust/crates/bh-discovery/src/lib.rs`.
- `BU_BROWSER_ID` and `BROWSER_USE_API_KEY` - remote Browser Use lifecycle cleanup inputs; read by `rust/bins/bhd/src/main.rs` and `rust/bins/bhctl/src/main.rs`.
- `BU_BROWSER_MODE`, `BU_DAEMON_IMPL`, `BU_REMOTE_TIMEOUT_MINUTES`, and `BU_LOCAL_DAEMON_WAIT_SECONDS` - smoke-runner scenario controls in `rust/bins/bhsmoke/src/main.rs`.
- `BU_RUST_DAEMON_BIN`, `BU_RUST_ADMIN_BIN`, and `BU_RUST_RUNNER_BIN` - binary override hooks in `rust/bins/bhctl/src/main.rs`, `rust/bins/bhrun/src/main.rs`, and `rust/bins/browser-harness-cli/src/main.rs`.
- `CARGO_HOME`, `CARGO`, `HOME`, `USERPROFILE`, `HOMEDRIVE`, `HOMEPATH`, and `PATH` - installer and binary discovery inputs in `rust/bins/browser-harness-cli/src/main.rs`.
- `.env.example` is present at `.env.example`; contents were not read because environment files are treated as secret-bearing.
- Rust workspace config: `rust/Cargo.toml`.
- Resolved dependencies: `rust/Cargo.lock`.
- CI config: `.github/workflows/ci.yml`.
- Install/bootstrap docs: `install.md`, `README.md`, and `rust/README.md`.
- No Node, Python package, Docker, or Go manifests were detected at repo root beyond the Rust workspace.
## Platform Requirements
- Install a stable Rust toolchain with Cargo; CI uses `dtolnay/rust-toolchain@stable` in `.github/workflows/ci.yml`.
- Add `wasm32-unknown-unknown` when building Rust guest crates under `rust/guests/rust-*`, per `rust/README.md`.
- Use a local Chrome, Chromium, or Microsoft Edge profile with remote debugging enabled for local browser verification, per `install.md` and `rust/crates/bh-discovery/src/lib.rs`.
- Use `ripgrep` for CI secret scanning when missing on Ubuntu runners, per `.github/workflows/ci.yml`.
- Deploy as native CLI binaries installed by `browser-harness install` into `$CARGO_HOME/bin` or a supplied root from `rust/bins/browser-harness-cli/src/main.rs`.
- Runtime connects either to a local Chrome/Edge CDP websocket discovered from `DevToolsActivePort` or to Browser Use cloud through `BU_CDP_WS`, `BU_BROWSER_ID`, and `BROWSER_USE_API_KEY`.
- The daemon IPC implementation is Unix-socket based in `rust/crates/bh-daemon/src/lib.rs` and `rust/bins/bhrun/src/main.rs`; treat Unix-like systems as the supported runtime target unless the daemon layer is ported.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Rust workspace code lives under `rust/`, with crate and binary package directories in kebab-case: `rust/crates/bh-wasm-host`, `rust/crates/bh-guest-sdk`, `rust/bins/browser-harness-cli`, `rust/guests/rust-github-trending`.
- Each Rust crate currently uses a flat source entry file rather than a multi-module tree: libraries use `src/lib.rs` such as `rust/crates/bh-protocol/src/lib.rs`; binaries use `src/main.rs` such as `rust/bins/bhrun/src/main.rs`.
- Rust package names use hyphens in `Cargo.toml`, while Rust import paths use underscores. Example: package `bh-wasm-host` in `rust/crates/bh-wasm-host/Cargo.toml` is imported as `bh_wasm_host` in `rust/bins/bhrun/src/main.rs`.
- Guest package directories use the `rust-{domain-or-scenario}` prefix and a `-guest` package name, for example `rust/guests/rust-spotify-search/Cargo.toml` and `rust/guests/rust-spotify-search/src/lib.rs`.
- Use Rust `snake_case` for functions and methods: `runtime_paths` in `rust/crates/bh-discovery/src/lib.rs`, `resolve_profile_name_in_profiles` in `rust/crates/bh-remote/src/lib.rs`, `wait_for_event_with_drain` in `rust/bins/bhrun/src/main.rs`.
- Command implementation functions mirror CLI command names using underscores: `current_tab`, `wait_for_load`, `wait_for_response`, and `handle_dialog` in `rust/bins/bhrun/src/main.rs`.
- Testable command helpers add dependency injection through suffixes such as `_with_sender` and `_with_drain`: `page_info_with_sender`, `goto_with_sender`, and `watch_events_collect_with_drain` in `rust/bins/bhrun/src/main.rs`.
- WASM guest crates export exactly one ABI entry point as `pub extern "C" fn run() -> i32`, then delegate to a private `run_inner()`: `rust/guests/rust-navigate-and-read/src/lib.rs`, `rust/guests/rust-github-trending/src/lib.rs`.
- Use `snake_case` for local variables and struct fields: `session_id`, `target_id`, `remote_browser_id` in `rust/crates/bh-daemon/src/lib.rs`; `daemon_name`, `guest_module`, `granted_operations` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use uppercase environment-variable strings and keep their exact names in errors and config paths: `BU_CDP_WS` in `rust/crates/bh-discovery/src/lib.rs`, `BROWSER_USE_API_KEY` in `rust/bins/bhctl/src/main.rs`, `BU_BROWSER_MODE` in `rust/bins/bhsmoke/src/main.rs`.
- Use `Value` for untyped JSON at browser/daemon boundaries, then convert to typed request and response structs as early as practical: `DaemonRequest` in `rust/crates/bh-protocol/src/lib.rs`, `RunnerConfig` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use `PascalCase` for structs and enums: `DaemonRequest` in `rust/crates/bh-protocol/src/lib.rs`, `BrowserUseClient` in `rust/crates/bh-remote/src/lib.rs`, `GuestServeRequest` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use semantic suffixes for protocol DTOs:
- Use `SCREAMING_SNAKE_CASE` for constants and protocol meta names: `PROTOCOL_VERSION`, `META_PAGE_INFO`, and `META_WAIT_FOR_LOAD` in `rust/crates/bh-protocol/src/lib.rs`; `DEFAULT_EVENT_CAPACITY` in `rust/crates/bh-daemon/src/lib.rs`.
## Code Style
- Format Rust code with:
- The formatting command is documented in `docs/development.md`; no custom `rustfmt.toml` or `.rustfmt.toml` is present, so use standard rustfmt defaults.
- The active Rust workspace uses edition `2021` through `[workspace.package]` in `rust/Cargo.toml`.
- No Clippy configuration file is detected: `clippy.toml` and `.clippy.toml` are not present.
- CI does not run `cargo clippy`; `.github/workflows/ci.yml` runs `cargo test --workspace --manifest-path rust/Cargo.toml`, CLI entry-point checks, install verification, and `scripts/scan_sensitive.sh`.
- Treat `cargo test --workspace --manifest-path rust/Cargo.toml` plus `./scripts/scan_sensitive.sh` as the enforced quality gate unless a future change adds Clippy to `.github/workflows/ci.yml`.
- Use `#[serde(default)]` for backward-compatible request fields and optional response fields: `DaemonRequest` in `rust/crates/bh-protocol/src/lib.rs`, `WaitForEventRequest` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use `skip_serializing_if = "Option::is_none"` for optional protocol fields so JSON output stays compact: `DaemonResponse` in `rust/crates/bh-protocol/src/lib.rs`, `CookieParam` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use explicit serde renames for external camelCase fields while keeping Rust fields snake_case, for example `#[serde(rename = "targetId")] pub target_id` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use internally tagged enums for protocol envelopes where the JSON discriminant is part of the wire format: `#[serde(tag = "command", rename_all = "snake_case")]` on `GuestServeRequest` and `#[serde(tag = "kind", rename_all = "snake_case")]` on `GuestServeResponse` in `rust/crates/bh-wasm-host/src/lib.rs`.
## Import Organization
- No custom Rust path aliases are configured.
- Use Cargo package names from `rust/Cargo.toml` and import them with Rust crate identifiers: `bh-protocol` becomes `bh_protocol`, `bh-wasm-host` becomes `bh_wasm_host`, and `bh-guest-sdk` becomes `bh_guest_sdk`.
- Guest crates import the SDK directly from `bh_guest_sdk`, for example `rust/guests/rust-event-waits-sdk/src/lib.rs` and `rust/guests/rust-producthunt-homepage/src/lib.rs`.
## Error Handling
- Application and CLI layers generally return `Result<T, String>` with contextual messages from `map_err` and `ok_or_else`: `request_json` in `rust/crates/bh-remote/src/lib.rs`, `send_raw` in `rust/crates/bh-cdp/src/lib.rs`, `read_json_stdin` in `rust/bins/bhctl/src/main.rs`.
- CLI `main` functions print a single user-facing error and exit non-zero: `rust/bins/browser-harness-cli/src/main.rs`, `rust/bins/bhctl/src/main.rs`, `rust/bins/bhrun/src/main.rs`.
- Prefer adding operation-specific context to errors:
- Use custom error types where the crate is a public API boundary. `GuestError` in `rust/crates/bh-guest-sdk/src/lib.rs` implements `Display` and `std::error::Error`.
- WASM guests use numeric exit codes at the ABI boundary. Keep detailed branching inside `run_inner() -> Result<(), i32>` and expose only `run() -> i32`, as in `rust/guests/rust-github-trending/src/lib.rs`.
- For non-critical CDP cleanup or best-effort event delivery, the code explicitly discards errors with `let _ = ...`: `mark_session`, `unmark_session`, and event sending paths in `rust/crates/bh-daemon/src/lib.rs` and `rust/crates/bh-cdp/src/lib.rs`.
## Logging
- CLI-facing errors use `eprintln!` in `main`: `rust/bins/browser-harness-cli/src/main.rs`, `rust/bins/bhctl/src/main.rs`, `rust/bins/bhrun/src/main.rs`.
- Daemon runtime logging writes to runtime log files derived from `RuntimePaths`: `runtime_paths` in `rust/crates/bh-discovery/src/lib.rs`, `log_line` and `log_tail` usage in `rust/crates/bh-daemon/src/lib.rs`.
- JSON-producing commands print machine-readable output to stdout and keep errors on stderr: `run` in `rust/bins/bhctl/src/main.rs`, `write_json` callers in `rust/bins/bhrun/src/main.rs`.
- Do not introduce a logging framework unless a module has a cross-cutting need; existing crates use plain `eprintln!`, JSON stdout, and daemon log files.
## Comments
- The Rust source contains very few line comments or doc comments. Prefer self-describing type and function names over explanatory comments for straightforward logic.
- Comments are appropriate for complex embedded browser scripts or protocol caveats, but keep them inside the nearest implementation file such as `rust/guests/rust-2048-autoplay/src/lib.rs`.
- User-facing and architectural explanations belong in Markdown docs such as `docs/development.md`, `docs/architecture.md`, and `rust/README.md`, not as long comments in `src/lib.rs` or `src/main.rs`.
- Not applicable; this repo is Rust-first.
- Rustdoc comments are not currently used as a dominant pattern in `rust/crates/*/src/lib.rs` or `rust/bins/*/src/main.rs`. Add Rustdoc only for new public APIs that need stable consumer guidance.
## Function Design
- Prefer typed request structs for commands and protocol operations: `CurrentTabRequest`, `ListTabsRequest`, `WaitForResponseRequest` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Prefer borrowing for simple string inputs: `stop_browser(&self, browser_id: &str)` in `rust/crates/bh-remote/src/lib.rs`, `goto(url: &str)` in `rust/crates/bh-guest-sdk/src/lib.rs`.
- Use generic closure parameters to make daemon, drain, and host-call boundaries testable without network or socket access: `current_tab_with_sender` and `wait_for_event_with_drain` in `rust/bins/bhrun/src/main.rs`.
- Use typed results for stable public APIs: `CurrentSessionResult`, `WaitForEventResult`, `WatchEventsLine` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Use `serde_json::Value` for dynamic browser/CDP payloads and protocol pass-throughs: `CdpClient::send_raw` in `rust/crates/bh-cdp/src/lib.rs`, `page_info_result` in `rust/crates/bh-daemon/src/lib.rs`.
- Use `Result<T, String>` for CLI-internal fallible helpers and attach context at the failure site: `build_install_report` in `rust/bins/browser-harness-cli/src/main.rs`, `daemon_launch_command` in `rust/bins/bhctl/src/main.rs`.
## Module Design
- Library crates expose their public API from `src/lib.rs`: `rust/crates/bh-protocol/src/lib.rs`, `rust/crates/bh-remote/src/lib.rs`, `rust/crates/bh-wasm-host/src/lib.rs`.
- Binary crates keep helpers private inside `src/main.rs` and expose behavior through CLI subcommands: `rust/bins/bhrun/src/main.rs`, `rust/bins/bhctl/src/main.rs`, `rust/bins/browser-harness-cli/src/main.rs`.
- `bh-guest-sdk` re-exports host protocol types with `pub use bh_wasm_host::{...}` in `rust/crates/bh-guest-sdk/src/lib.rs`; keep SDK consumers on `bh_guest_sdk` rather than importing host internals directly.
- No separate barrel modules are present. `src/lib.rs` is the crate API root for each library.
- No `mod`-split source tree is currently used under `rust/crates/*/src/`; adding a new module should be paired with a clear responsibility boundary and a `mod` declaration in that crate’s `src/lib.rs`.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## System Overview
```text
```
## Component Responsibilities
| Component | Responsibility | File |
|-----------|----------------|------|
| Top-level `browser-harness` CLI | Routes public commands to `bhctl` for admin/lifecycle or `bhrun` for helper/runner commands; owns `install` and `verify-install`. | `rust/bins/browser-harness-cli/src/main.rs` |
| `bhctl` admin CLI | Creates/lists/stops Browser Use cloud browsers, resolves/syncs profiles through `profile-use`, starts/stops/restarts the daemon, and reports daemon liveness. | `rust/bins/bhctl/src/main.rs` |
| `bhrun` runner CLI | Parses typed JSON command payloads, sends daemon meta/CDP requests, implements wait/http utilities, executes WASM guests, and serves persistent guest sessions. | `rust/bins/bhrun/src/main.rs` |
| `bhd` daemon binary | Reads daemon environment, initializes runtime files, starts `bh-daemon::serve`, stops remote cloud browsers on shutdown, and cleans pid/socket files. | `rust/bins/bhd/src/main.rs` |
| Daemon runtime | Owns CDP client, active session/target state, dialog/event buffer, Unix socket server, helper meta command handlers, and raw CDP forwarding. | `rust/crates/bh-daemon/src/lib.rs` |
| CDP client | Maintains the WebSocket connection, assigns CDP ids, maps responses to pending callers, emits CDP events on a Tokio channel, and identifies browser-level methods. | `rust/crates/bh-cdp/src/lib.rs` |
| Discovery/runtime paths | Resolves `/tmp/bu-<name>.*` runtime paths, discovers local Chrome/Edge `DevToolsActivePort`, filters internal browser URLs, and honors `BU_CDP_WS`. | `rust/crates/bh-discovery/src/lib.rs` |
| Wire protocol | Defines daemon JSON-line request/response shapes and all daemon meta command names. | `rust/crates/bh-protocol/src/lib.rs` |
| Remote Browser Use client | Wraps Browser Use REST APIs for browser lifecycle, CDP websocket URL resolution, and cloud profile lookup. | `rust/crates/bh-remote/src/lib.rs` |
| WASM host contract | Defines runner config, guest serve protocol, operation request/response DTOs, host manifest, event filters, and operation metadata. | `rust/crates/bh-wasm-host/src/lib.rs` |
| Guest SDK | Provides Rust guest helper functions over the imported `bh.call_json` host function. | `rust/crates/bh-guest-sdk/src/lib.rs` |
| Smoke runner | Exercises CLI, daemon, browser actions, tabs, events, guests, remote mode, and site-specific guest workflows. | `rust/bins/bhsmoke/src/main.rs` |
| Domain knowledge | Stores durable site-specific selectors, APIs, waits, traps, and optional guest mapping. | `domains/<site>/skill.md`, `domains/README.md` |
| Interaction knowledge | Stores reusable browser mechanics such as tabs, screenshots, downloads, network waits, dialogs, uploads, scrolling, and viewport rules. | `interaction-skills/*.md` |
## Pattern Overview
- Keep `browser-harness` thin. New browser operation commands belong in `bhrun` and daemon meta handling, while browser lifecycle/admin commands belong in `bhctl`.
- Keep `bhd` as the only owner of the live browser websocket and mutable browser session state.
- Use `bh-protocol` constants for meta command names instead of string literals spread across crates.
- Use typed request/response structs in `bh-wasm-host` for public helper and guest surfaces.
- Use guests for packaged repeatable workflows; guests call host operations and never connect to the daemon socket directly.
- Keep site-specific knowledge in `domains/<site>/skill.md`; Rust guests are optional packaged workflows, not the default artifact for every domain.
## Layers
- Purpose: Teaches agents how to use the harness and site-specific browser workflows.
- Location: `SKILL.md`, `domains/`, `interaction-skills/`, `docs/`.
- Contains: domain `skill.md` files, reusable interaction mechanics, architecture/development notes, install guide.
- Depends on: the public CLI surface exposed by `browser-harness`, `bhctl`, `bhrun`, and guest examples under `rust/guests/`.
- Used by: humans and agents before invoking or extending runtime code.
- Purpose: Provides the stable installed command `browser-harness`.
- Location: `rust/bins/browser-harness-cli/src/main.rs`.
- Contains: route tables (`ADMIN_COMMANDS`, `RUNNER_HELP`), process spawning, install/verify-install implementation.
- Depends on: child binaries `bhctl`, `bhrun`, `bhd`; `cargo` fallback; filesystem checks for install verification.
- Used by: all external shell/subprocess callers and documentation examples.
- Purpose: Handles daemon lifecycle, cloud browser lifecycle, and profile sync/lookup.
- Location: `rust/bins/bhctl/src/main.rs`, `rust/crates/bh-remote/src/lib.rs`.
- Contains: `create-browser`, `list-browsers`, `stop-browser`, `ensure-daemon`, `restart-daemon`, `list-cloud-profiles`, profile-use wrappers.
- Depends on: `bh-daemon` runtime file helpers, `bh-remote` Browser Use API client, `profile-use` external command.
- Used by: `browser-harness` admin command route and remote browser setup flows.
- Purpose: Provides typed browser operations, wait utilities, HTTP utility, raw CDP escape hatch, and WASM guest execution.
- Location: `rust/bins/bhrun/src/main.rs`, `rust/crates/bh-wasm-host/src/lib.rs`.
- Contains: command dispatch, request normalization, `send_daemon_request`, event wait loops, `GuestRuntime`, capability checks, manifest/capabilities output.
- Depends on: `bh-protocol` daemon messages, `bh-wasm-host` DTOs, `wasmtime`, `reqwest`, Unix sockets.
- Used by: direct `bhrun` calls, `browser-harness` runner command route, Rust/WAT guests through host calls.
- Purpose: Owns the single live CDP connection and browser state for a daemon namespace.
- Location: `rust/bins/bhd/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`.
- Contains: Unix socket listener, per-connection JSON-line handler, active session attachment, event buffer, dialog tracking, helper implementations, raw CDP forwarding, runtime file lifecycle.
- Depends on: `bh-cdp`, `bh-discovery`, `bh-protocol`, `bh-remote`, Tokio.
- Used by: `bhrun` helper calls and guest host calls.
- Purpose: Provides asynchronous WebSocket request/response and event demultiplexing for Chrome DevTools Protocol.
- Location: `rust/crates/bh-cdp/src/lib.rs`.
- Contains: `CdpClient`, `CdpEvent`, pending-response map, reader task, `send_raw`, browser-level method classifier.
- Depends on: `tokio-tungstenite`, `futures-util`, `serde_json`, Tokio channels.
- Used by: `bh-daemon` only.
- Purpose: Resolves CDP endpoints and daemon runtime file paths.
- Location: `rust/crates/bh-discovery/src/lib.rs`.
- Contains: `RuntimePaths`, `/tmp/bu-<name>.sock|pid|log`, local Chrome/Edge profile search, `BU_CDP_WS` override, internal URL filtering.
- Depends on: filesystem, TCP probing, environment variables.
- Used by: `bh-daemon` and indirectly by `bhctl` daemon lifecycle commands.
- Purpose: Prevents drift between daemon, runner, and guest surfaces.
- Location: `rust/crates/bh-protocol/src/lib.rs`, `rust/crates/bh-wasm-host/src/lib.rs`.
- Contains: `DaemonRequest`, `DaemonResponse`, `META_*` constants, `RunnerConfig`, operation DTOs, event filters, host manifest.
- Depends on: `serde`, `serde_json`.
- Used by: `bh-daemon`, `bhrun`, `bh-guest-sdk`, tests, guests.
- Purpose: Packages repeatable workflow logic as WASM modules.
- Location: `rust/guests/*.wat`, `rust/guests/rust-*/src/lib.rs`, `rust/crates/bh-guest-sdk/src/lib.rs`.
- Contains: exported `run()` entry points, site/workflow constants, JS extraction scripts, SDK calls, guest exit-code checks.
- Depends on: `bh-guest-sdk`, `serde`, `serde_json`; host capabilities granted by `RunnerConfig`.
- Used by: `bhrun run-guest`, `bhrun serve-guest`, and `bhsmoke` guest scenarios.
- Purpose: Provides repository acceptance and smoke coverage for local/remote runtime behavior.
- Location: `rust/bins/bhsmoke/src/main.rs`, `.github/workflows/ci.yml`, `scripts/scan_sensitive.sh`.
- Contains: smoke scenario orchestration, cargo test workflow, install verification, sensitive content scan.
- Depends on: installed or repo-local CLI binaries, live local/remote browser settings for smoke paths.
- Used by: local development and CI.
## Data Flow
### Primary Runner Command Path
- User/agent invokes `browser-harness page-info` or another runner command (`rust/bins/browser-harness-cli/src/main.rs`).
- `route_command` classifies non-admin commands as runner commands and `spawn_child` starts sibling `bhrun` or `cargo run --bin bhrun` fallback (`rust/bins/browser-harness-cli/src/main.rs`).
- `bhrun` parses stdin JSON into a typed request such as `PageInfoRequest` and normalizes defaults (`rust/bins/bhrun/src/main.rs`, `rust/crates/bh-wasm-host/src/lib.rs`).
- The runner builds `DaemonRequest { meta: Some(META_PAGE_INFO), params: ... }` and calls `send_daemon_request` over `/tmp/bu-<daemon_name>.sock` (`rust/bins/bhrun/src/main.rs`).
- `bh-daemon::handle_stream` parses one JSON line into `DaemonRequest` and calls `Daemon::handle_request` (`rust/crates/bh-daemon/src/lib.rs`).
- `Daemon::page_info_result` evaluates JavaScript in the current session through `Runtime.evaluate` via `send_with_retry` and `CdpClient::send_raw` (`rust/crates/bh-daemon/src/lib.rs`, `rust/crates/bh-cdp/src/lib.rs`).
- `DaemonResponse { result }` is serialized as one JSON line and returned to `bhrun`, which deserializes/prints pretty JSON to stdout (`rust/crates/bh-protocol/src/lib.rs`, `rust/bins/bhrun/src/main.rs`).
### Daemon Startup Path
- User/agent invokes `browser-harness ensure-daemon` (`rust/bins/browser-harness-cli/src/main.rs`).
- The facade routes `ensure-daemon` to `bhctl` because it is listed in `ADMIN_COMMANDS` (`rust/bins/browser-harness-cli/src/main.rs`).
- `bhctl` parses optional JSON `{name, wait, env}`, checks `already_running`, and launches `bhd` using `BU_RUST_DAEMON_BIN`, sibling binary, or `cargo run --bin bhd` fallback (`rust/bins/bhctl/src/main.rs`).
- `bhd` builds `DaemonConfig` from `BU_NAME`, `BU_BROWSER_ID`, and `BROWSER_USE_API_KEY`, writes pid/log files, and calls `serve` (`rust/bins/bhd/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`).
- `serve` binds `/tmp/bu-<name>.sock`, sets socket mode `0600`, resolves the CDP endpoint through `get_ws_url`, connects `CdpClient`, attaches the first real page, and starts the CDP event loop (`rust/crates/bh-daemon/src/lib.rs`, `rust/crates/bh-discovery/src/lib.rs`, `rust/crates/bh-cdp/src/lib.rs`).
- `bhctl` polls until the socket accepts connections, then returns a JSON status report (`rust/bins/bhctl/src/main.rs`).
### Remote Browser Control Path
- User/agent invokes `browser-harness create-browser` with JSON payload (`rust/bins/browser-harness-cli/src/main.rs`).
- The facade routes the command to `bhctl`; `bhctl` creates `BrowserUseClient` from `BROWSER_USE_API_KEY` (`rust/bins/bhctl/src/main.rs`).
- `normalize_create_browser_payload` resolves `profileName` into `profileId` when present (`rust/bins/bhctl/src/main.rs`, `rust/crates/bh-remote/src/lib.rs`).
- `BrowserUseClient::create_browser` posts to Browser Use, then `cdp_ws_from_url` reads `/json/version` to derive `cdpWsUrl` (`rust/crates/bh-remote/src/lib.rs`).
- Caller starts a daemon with `BU_CDP_WS` and `BU_BROWSER_ID` in `ensure-daemon` env. On shutdown, `bhd` calls `stop_remote` so Browser Use cloud billing/session ends cleanly (`rust/bins/bhd/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`).
### WASM Guest Run Path
- User/agent invokes `bhrun run-guest <path>` or `browser-harness run-guest <path>` with `RunnerConfig` JSON (`rust/bins/bhrun/src/main.rs`).
- `GuestRuntime::new` loads `.wat` or `.wasm` with Wasmtime, registers the imported host function `bh.call_json`, instantiates the module, and locates exported `run` (`rust/bins/bhrun/src/main.rs`).
- Rust guests call SDK helpers such as `goto`, `wait_for_load`, `js`, or `wait_for_response`; each helper serializes an operation name and request through `call_json` (`rust/crates/bh-guest-sdk/src/lib.rs`).
- The host `dispatch_guest_operation` checks `granted_operations` plus `allow_http`/`allow_raw_cdp` gates, injects `daemon_name` when missing, dispatches to the same runner helper functions, and records `GuestCallRecord` entries (`rust/bins/bhrun/src/main.rs`, `rust/crates/bh-wasm-host/src/lib.rs`).
- `invoke_run` returns `GuestRunResult { exit_code, success, calls, trap }`; `serve-guest` keeps the same runtime and invocation count across NDJSON `run` commands (`rust/bins/bhrun/src/main.rs`).
### Event Wait Path
- The daemon enables Page, DOM, Runtime, Network, Log, and Console domains for each active session (`rust/crates/bh-daemon/src/lib.rs`).
- `CdpClient` forwards CDP events to a Tokio channel; `run_event_loop` calls `Daemon::handle_event` (`rust/crates/bh-cdp/src/lib.rs`, `rust/crates/bh-daemon/src/lib.rs`).
- `handle_event` appends normalized events `{method, params, session_id}` into a bounded `VecDeque` and tracks dialog open/close state (`rust/crates/bh-daemon/src/lib.rs`).
- Runner waits (`wait-for-event`, `wait-for-response`, `wait-for-console`, `watch-events`) repeatedly drain events via `META_DRAIN_EVENTS` and match them with filters from `bh-wasm-host` (`rust/bins/bhrun/src/main.rs`, `rust/crates/bh-wasm-host/src/lib.rs`).
- Daemon namespace state is keyed by `BU_NAME`, with socket/pid/log at `/tmp/bu-<name>.sock`, `/tmp/bu-<name>.pid`, and `/tmp/bu-<name>.log` (`rust/crates/bh-discovery/src/lib.rs`).
- Live browser state is process-local in `DaemonState { session_id, target_id, dialog, events }`, protected by `Arc<Mutex<_>>` (`rust/crates/bh-daemon/src/lib.rs`).
- Guest state lives inside Wasmtime `Store<GuestHostState>` for one `run-guest` invocation or across multiple invocations in `serve-guest` when `persistent_guest_state=true` (`rust/bins/bhrun/src/main.rs`).
- Site knowledge state is plain Markdown under `domains/` and `interaction-skills/`; it does not mutate at runtime.
## Key Abstractions
- Purpose: Stable IPC between `bhrun` and `bhd`.
- Examples: `DaemonRequest`, `DaemonResponse`, `META_PAGE_INFO`, `META_GOTO`, `META_DRAIN_EVENTS` in `rust/crates/bh-protocol/src/lib.rs`.
- Pattern: one JSON line request over Unix socket, one JSON line response.
- Purpose: Allows independent browser sessions/daemons for parallel agents or remote browsers.
- Examples: `RuntimePaths` in `rust/crates/bh-discovery/src/lib.rs`, `DaemonConfig` in `rust/crates/bh-daemon/src/lib.rs`.
- Pattern: `BU_NAME` defaults to `default` and maps to `/tmp/bu-<name>.sock|pid|log`.
- Purpose: Converts concurrent Rust calls into CDP request ids and asynchronous response/event streams.
- Examples: `CdpClient`, `CdpEvent`, `send_raw` in `rust/crates/bh-cdp/src/lib.rs`.
- Pattern: WebSocket writer protected by a mutex; pending id map resolves oneshot responses; events flow through `mpsc`.
- Purpose: Provide ergonomic browser helpers without exposing every caller to raw CDP.
- Examples: `META_GOTO`, `META_JS`, `META_CLICK`, `META_SCREENSHOT`, `META_WAIT_FOR_LOAD` in `rust/crates/bh-protocol/src/lib.rs`; handlers in `Daemon::handle_request` in `rust/crates/bh-daemon/src/lib.rs`.
- Pattern: `request.meta` selects helper; raw `request.method` path remains available for unsupported CDP calls.
- Purpose: Define public JSON command and guest operation shapes once.
- Examples: `GotoRequest`, `JsRequest`, `WaitForEventRequest`, `SetViewportRequest`, `CookieParam` in `rust/crates/bh-wasm-host/src/lib.rs`.
- Pattern: `serde` structs with `Default` and `normalized()` methods for CLI input defaults.
- Purpose: Restricts what a guest can do at runtime.
- Examples: `RunnerConfig { granted_operations, allow_http, allow_raw_cdp, persistent_guest_state }` in `rust/crates/bh-wasm-host/src/lib.rs`; checks in `dispatch_guest_operation` in `rust/bins/bhrun/src/main.rs`.
- Pattern: allow-list by operation name plus explicit extra gates for HTTP and raw CDP.
- Purpose: Lets Rust guests use browser operations without hand-writing host import serialization.
- Examples: `goto`, `js`, `wait_for_response`, `watch_events`, `screenshot` in `rust/crates/bh-guest-sdk/src/lib.rs`.
- Pattern: each helper calls generic `call_json(operation, request)` and deserializes typed responses.
- Purpose: Holds durable per-site workflow knowledge and maps some sites to guest crates.
- Examples: `domains/github/skill.md`, `domains/reddit/skill.md`, `domains/spotify/skill.md`, `domains/README.md`.
- Pattern: knowledge-first Markdown with optional Rust guest under `rust/guests/rust-<site-or-workflow>/`.
## Entry Points
- Location: `rust/bins/browser-harness-cli/src/main.rs`.
- Triggers: `browser-harness <command>` from shell, agent tool, or thin subprocess wrapper.
- Responsibilities: route admin vs runner commands; install/verify installed Rust-only binaries.
- Location: `rust/bins/bhctl/src/main.rs`.
- Triggers: `bhctl ...` directly or `browser-harness` admin route.
- Responsibilities: Browser Use API calls, profile-use integration, daemon lifecycle.
- Location: `rust/bins/bhrun/src/main.rs`.
- Triggers: `bhrun ...` directly or `browser-harness` runner route.
- Responsibilities: JSON stdin parsing, typed helper calls, waits, HTTP GET, raw CDP, guest runtime.
- Location: `rust/bins/bhd/src/main.rs`.
- Triggers: launched by `bhctl ensure-daemon`, manually run, or sibling installed binary.
- Responsibilities: process-level daemon setup and cleanup around `bh-daemon::serve`.
- Location: `rust/crates/bh-daemon/src/lib.rs`.
- Triggers: `bhd` calls `serve`.
- Responsibilities: bind socket, connect CDP, attach page, accept IPC, handle requests/events.
- Location: `rust/bins/bhsmoke/src/main.rs`.
- Triggers: local `cargo run --bin bhsmoke -- <scenario>` or CI/manual checks.
- Responsibilities: acceptance checks for runtime, CLI, guest, event, tab, remote, and site-specific workflows.
- Location: `rust/guests/rust-*/src/lib.rs`.
- Triggers: `bhrun run-guest` or `bhrun serve-guest` loading compiled `.wasm` artifacts.
- Responsibilities: workflow-specific automation with exported `run() -> i32`.
## Architectural Constraints
- **Threading:** `bhd`, `bhctl`, and CDP transport use Tokio; `bhrun` is mostly synchronous and opens a blocking Unix stream per daemon call, but creates a current-thread Tokio runtime for `http_get`.
- **IPC shape:** Daemon IPC is Unix-socket-only and line-oriented. Windows named pipes are not represented in the current runtime code (`rust/bins/bhrun/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`).
- **Global state:** Runtime state is daemon-process local in `DaemonState`; runtime files are global `/tmp/bu-<name>.*`; command defaults use environment variables such as `BU_NAME`, `BU_CDP_WS`, `BU_BROWSER_ID`, `BROWSER_USE_API_KEY`, `BU_RUST_*`.
- **Session ownership:** Only `bhd` owns CDP sessions. `bhrun`, guests, Python subprocess wrappers, and domain skills must call the CLI/daemon protocol rather than owning websocket state.
- **Event buffer:** `META_DRAIN_EVENTS` is destructive; wait/watch operations consume the daemon event queue. Start waits before triggering an action for reliable matching (`interaction-skills/network-requests.md`, `rust/bins/bhrun/src/main.rs`).
- **Guest boundary:** Guests cannot bypass host checks. `http_get` requires both `granted_operations` and `allow_http`; `cdp_raw` requires both `granted_operations` and `allow_raw_cdp` (`rust/bins/bhrun/src/main.rs`).
- **Local browser discovery:** Local attach reads `DevToolsActivePort` from known Chrome/Edge profile directories and waits for the port; it does not launch a browser (`rust/crates/bh-discovery/src/lib.rs`).
- **Cloud browser shutdown:** Remote cleanup depends on `BU_BROWSER_ID` and `BROWSER_USE_API_KEY` being present when `bhd` exits (`rust/bins/bhd/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`).
- **Circular imports:** No Rust crate cycle is present in `rust/Cargo.toml`; dependency flow is one-way: binaries -> crates, `bh-daemon` -> `bh-cdp`/`bh-discovery`/`bh-protocol`/`bh-remote`, `bh-guest-sdk` -> `bh-wasm-host` -> `bh-protocol`.
- **File size concentration:** `rust/bins/bhsmoke/src/main.rs`, `rust/bins/bhrun/src/main.rs`, `rust/crates/bh-wasm-host/src/lib.rs`, `rust/crates/bh-daemon/src/lib.rs`, and `rust/crates/bh-guest-sdk/src/lib.rs` contain many responsibilities in single files; when adding code, prefer extracting by responsibility instead of expanding these files further.
## Anti-Patterns
### Adding a second runtime manager
### Putting runner helpers in the top-level CLI
### Letting guests talk to sockets or CDP directly
### Treating domain docs as executable runtime
### Reading `.env` or secrets into docs
## Error Handling
- Use `map_err(|err| format!("context: {err}"))` around filesystem, process, JSON, network, and CDP operations (`rust/bins/bhrun/src/main.rs`, `rust/bins/bhctl/src/main.rs`, `rust/crates/bh-daemon/src/lib.rs`).
- Daemon helper handlers return `DaemonResponse { error: Some(err) }` rather than panicking (`rust/crates/bh-daemon/src/lib.rs`).
- `send_with_retry` re-attaches once when the current CDP session is stale (`rust/crates/bh-daemon/src/lib.rs`).
- Guest host-call failures set `GuestHostState.error` and return `-1`; `GuestRunResult.trap` carries the failure (`rust/bins/bhrun/src/main.rs`).
- Live smoke and remote operations return JSON status objects or `String` errors rather than throwing exceptions (`rust/bins/bhsmoke/src/main.rs`, `rust/crates/bh-remote/src/lib.rs`).
## Cross-Cutting Concerns
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Source: [yangchuansheng/browser-harness-rust](https://github.com/yangchuansheng/browser-harness-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
