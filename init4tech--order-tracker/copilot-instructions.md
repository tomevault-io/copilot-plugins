## order-tracker

> **Keep this file and the readme up to date.** After any change to the repo (new files, renamed modules, added dependencies, changed conventions, etc.), update the relevant sections of this document and the README before finishing.

# Signet Tracker

**Keep this file and the readme up to date.** After any change to the repo (new files, renamed modules, added dependencies, changed conventions, etc.), update the relevant sections of this document and the README before finishing.

Cargo workspace for tracking the lifecycle and diagnosing fill failures of Signet orders, plus an HTTP/WebSocket API server for subscribing to order status updates.

## Project Structure

```
Cargo.toml - Workspace root (members: crates/*)

crates/tracker/ - signet-tracker: tracking library (on-chain queries via Provider trait)
  src/lib.rs - Library root, module exports
  src/error.rs - Error enum (manual Display with alt-formatting source chain, manual Error impl)
  src/order_status.rs - OrderStatus enum (Pending/Filled/Expired with embedded order_hash + diagnostics), FillInfo, FillOutput, Chain
  src/order_diagnostics.rs - OrderDiagnostics, DeadlineCheck, AllowanceChecks, BalanceChecks, MaybeBool
  src/order_tracker.rs - OrderTracker<RuP, HostP>: status(), status_for_order(), check_*, find_fill()
  src/fill_search.rs - Filled event scanning, output matching, ERC-20 balance/allowance queries
  src/amount.rs - Amount wrapper (U256 + decimal string serialization)
  src/timestamp.rs - Timestamp wrapper (unix secs + ISO 8601 serialization via jiff)
  src/pretty_duration.rs - PrettyDuration wrapper (ISO 8601 duration serialization via jiff)
  src/token_symbol_cache.rs - RwLock cache resolving token addresses to symbols (constants + on-chain fallback)

crates/tracker-server/ - signet-tracker-server: HTTP/WS API server (lib + bin)
  src/lib.rs - Server library root, run(), handle_signals(), config_from_env(), TasksJoinHandles, handle_task_exit()
  src/main.rs - Binary entry point
  src/config.rs - Config (FromEnv + Init4Config), provider type aliases, connect methods with retry
  src/service.rs - Axum HTTP/WS router, AppState, serve_tracker()
  src/metrics.rs - Prometheus metrics with typed label enums
  src/ingestion/
    block_watcher.rs - Rollup WS block subscription + host HTTP block polling (BlockTip, BlockNumbers)
    event_watcher.rs - Rollup WS subscription to Filled + Order log events
    order_discovery.rs - Periodic tx-cache polling for new orders
  src/state/
    event_store.rs - In-memory BTreeMap of recent Filled/Order events with block-based pruning
    tracked_order.rs - TrackedOrder wrapper (SignedOrder + OrderStatus + deadline)
    state_manager.rs - Central select! loop: processes events, tracks orders, broadcasts updates
  src/ws/
    handlers.rs - Axum WebSocket upgrade handlers for single-order and all-orders endpoints
    messages.rs - OrderFilter, StatusFilter for WS client filtering
```

## Build & Run

- **Rust edition**: 2024, MSRV 1.88
- **Build**: `cargo build`
- **Test**: `cargo t`
- **Lint**: `cargo clippy --all-features --all-targets` and `cargo clippy --no-default-features --all-targets`
- **Formatting**: `cargo +nightly fmt` (uses `rustfmt.toml` with `reorder_imports`, `use_field_init_shorthand`, `use_small_heuristics = "Max"`)

## Key Dependencies

### signet-tracker (library)
- **signet-sdk crates** (`signet-constants`, `signet-orders`, `signet-tx-cache`, `signet-types`, `signet-zenith`): Signet chain types, order signing, Permit2 nonce checks, tx-cache client
- **alloy**: Ethereum provider/types, contract calls, event log queries
- **jiff**: Human-readable timestamp and duration serialization
- **tracing**: Instrumentation
- **serde**: Serializable output types for consumer rendering

### signet-tracker-server (API server)
- **signet-tracker**: The tracking library
- **init4-bin-base**: Config loading (FromEnv derive), tracing/metrics init, provider config types
- **alloy**: RPC providers (HTTP + WS)
- **axum**: HTTP + WebSocket server
- **backon**: Exponential backoff retry for provider/tx-cache connections
- **tokio** + **tokio-util**: Async runtime, signal handling, CancellationToken

## Conventions

- All output types implement `Serialize` so consumers can render as JSON
- `OrderStatus` is the single output type — each variant (Pending/Filled/Expired) contains the order_hash and relevant fields for that state
- `OrderTracker` is generic over `RuP: Provider` and `HostP: Provider` — no concrete provider types
- Public API: `status()` (fetches from tx-cache), `status_for_order()` (takes existing order), `check_*` methods, `find_fill()`
- Workspace dependencies are declared in the root `Cargo.toml` and inherited via `.workspace = true`
- No global static config — `config_from_env()` returns the config and OTLP guard; the binary owns them on the stack
- Provider connections use exponential backoff retry (following the pattern from `init4/filler`)
- Graceful shutdown via `CancellationToken` cancelled on SIGINT/SIGTERM; all tasks use `handle_task_exit()` for consistent error reporting
- Visibility is as restrictive as possible — `pub(crate)` preferred over `pub` for internal items
- Metrics use typed label enums rather than arbitrary strings
- `Error` uses manual `Display` impl: normal format shows the message, alternate format (`{:#}`) walks the full source chain

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/init4tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
