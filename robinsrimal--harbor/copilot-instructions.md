## harbor

> These instructions are for agents working on Harbor's Rust codebase, primarily

# Harbor Rust Agent Guide

## Scope

These instructions are for agents working on Harbor's Rust codebase, primarily
under `core/` and `raim/src-tauri/`.

- Preserve the project's existing style, architecture, MSRV, and dependency
  choices unless asked to change them.
- Prefer correctness, maintainability, and clear reasoning over cleverness.
- Read the local code before making assumptions.
- Read the local design docs before changing protocol, storage, or runtime
  behavior.

## Project Defaults

For existing projects:

- Do not bump `rust-version`, edition, or major dependencies without asking.
- Match the repo's current formatting, linting, and test patterns.

For new Rust projects, prefer:

```toml
[package]
edition = "2024"
rust-version = "1.85"

[lints.rust]
unsafe_code = "warn"

[lints.clippy]
all = "warn"
pedantic = "warn"
```

## Rust Style

- Use `snake_case` for functions and variables.
- Use `PascalCase` for types and traits.
- Use `SCREAMING_SNAKE_CASE` for constants.
- Prefer `rustfmt` output over hand formatting.
- Keep comments brief and useful.
- Prefer descriptive lifetime names like `'src` or `'ctx` when that improves
  readability.

## Error Handling

- In library code, prefer `Result` and `?` over `unwrap()`.
- Use `Option` when absence is a normal outcome.
- Use `panic!` or `expect()` only for true invariants or startup assumptions.
- Prefer typed errors in libraries, usually with `thiserror`.
- Prefer ergonomic propagation in applications, usually with `anyhow`.
- Add context when propagation alone would hide what failed.

## Ownership And API Design

- Prefer borrowing or redesigning ownership before reaching for `clone()`.
- Use newtypes when domain meaning matters.
- Keep lifetimes and borrowing structure simple and explicit.
- Avoid interior mutability unless it solves a real design constraint.

## Concurrency And Async

- CPU-bound work usually wants threads or Rayon.
- I/O-bound work usually wants async.
- Prefer message passing when shared mutable state is not necessary.
- Use `Arc<T>` for immutable shared state.
- Use `Arc<Mutex<T>>` or `Arc<RwLock<T>>` only when shared mutation is truly
  needed.
- Do not hold locks across `.await`.
- Do not block async executors with synchronous sleep or heavy CPU work.

## Unsafe Rust

- Keep `unsafe` blocks as small as possible.
- Every `unsafe` block must include a `// SAFETY:` comment explaining the
  invariants.
- Every `unsafe fn` must document a `# Safety` section.
- Prefer safe wrappers around unsafe internals.
- Treat manual `Send`/`Sync`, pointer arithmetic, `transmute`, `from_raw_parts`,
  and FFI boundaries as high-scrutiny code.

## Review Priorities

When asked to review code, prioritize:

1. Bugs and behavioral regressions.
2. Unsoundness, panic hazards, and bad error handling.
3. Concurrency mistakes such as lock ordering, deadlocks, and holding guards
   across `.await`.
4. Missing tests for edge cases and failure paths.
5. Clarity and maintainability.

Present findings first, ordered by severity, with file references when
possible. Keep summaries brief.

## Testing

- Add or update tests when behavior changes.
- Prefer focused unit tests for logic and integration tests for boundaries.
- To run the Harbor simulation targets under `core/simulation/`, use the
  simulation-only command below. It normally runs without elevated
  permissions. Environment-dependent checks, such as deployed-bootstrap and
  LAN mDNS tests, are included in the suite but ignored by default unless
  explicitly unignored. Keep the `bootstrap-sim` feature scoped to the local
  bootstrap-chain test; do not enable it for the whole simulation suite unless
  you are working on that specific path.
- To run one individual simulation target, use `cargo test -p harbor --test
  <scenario_name>`, where `<scenario_name>` matches the scenario file name
  without the `.rs` suffix. Add `-- --nocapture` when you want log output in
  the terminal while the scenario runs. Add `-- --ignored` for an ignored
  environment-dependent simulation.

```bash
cargo test -p harbor --tests
cargo test -p harbor --test runtime_background_tasks_start -- --nocapture
cargo test -p harbor --test topics_send_online -- --nocapture
cargo test -p harbor --test lan_mdns_smoke -- --ignored --nocapture
HARBOR_LAN_LAYERED_SOAK_SECS=60 cargo test -p harbor --test lan_layered_soak -- --ignored --nocapture
cargo test -p harbor --features bootstrap-sim --test bootstrap_three_node_chain_bootstrap_then_topic_send
```

- For unsafe or concurrent code, test edge cases and failure paths explicitly.
- If runtime or simulation tests need elevated permissions for network socket
  binding, request elevation and rerun them rather than treating bind failures
  as product failures.
- If you could not run tests, say so clearly.

## Practical Heuristics

- `Box<T>` for owned heap allocation.
- `Rc<T>` for shared ownership on one thread.
- `Arc<T>` for shared ownership across threads.
- `RefCell<T>` for single-threaded interior mutability.
- `Mutex<T>` or `RwLock<T>` for synchronized shared mutation.
- `OnceLock` or `LazyLock` over older lazy-init patterns when MSRV allows.

## Working Style

- Make the smallest correct change that solves the problem.
- Do not rewrite working code without a clear reason.
- Explain tradeoffs when there are multiple reasonable designs.
- After each new feature or refactor, check whether design docs need updates
  and update them when behavior, ownership, or architecture changed.
- If a change has hidden risk, pause and surface it before committing.

---
> Source: [RobinSrimal/harbor](https://github.com/RobinSrimal/harbor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
