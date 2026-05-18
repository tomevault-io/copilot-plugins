## gatehouse

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Commands

```bash
cargo build                                                    # build
cargo fmt --all                                                # format
cargo clippy --all-targets --all-features -- -D warnings       # lint (CI uses -D warnings)
cargo test --all-targets --all-features                        # all tests
cargo test <test_name>                                         # single test
cargo bench                                                    # criterion benchmarks
cargo run --example axum                                       # HTTP server on :8000
cargo run --example actix_web                                  # HTTP server on :8080
```

CI order: build → clippy → doc → test. Always `fmt` + `clippy` before committing.

## Architecture

Single-crate library, all in `src/lib.rs`. Unit tests co-located under `#[cfg(test)]`; integration tests in `tests/` include examples via `include!(concat!(env!("CARGO_MANIFEST_DIR"), "/examples/..."))`.

Core abstraction is `Policy<S, R, A, C>` trait (async `evaluate_access()` → `PolicyEvalResult`). `PermissionChecker` aggregates policies with OR semantics (short-circuits on first allow). `PolicyBuilder` chains predicates with AND logic.

Built-in policies: `RbacPolicy`, `AbacPolicy`, `RebacPolicy`. Combinators: `AndPolicy`, `OrPolicy`, `NotPolicy`.

All policy closures require `Send + Sync + 'static`. No `Result` error types — decisions are always `PolicyEvalResult` variants (`Granted`, `Denied`, `Combined`). Telemetry via `tracing` with OpenTelemetry semantic conventions.

---
> Source: [thepartly/gatehouse](https://github.com/thepartly/gatehouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
