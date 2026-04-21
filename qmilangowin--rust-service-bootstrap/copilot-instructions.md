## rust-service-bootstrap

> Rust service bootstrap project rules

<!--
This file is intentionally committed to the repository.
Anyone cloning this bootstrap gets Cursor AI context out of the box — no setup required.
-->

# Rust Service Bootstrap — Cursor Rules

<!--
Global Rust working-style rules are embedded here so the project is self-contained.
No additional setup required beyond opening this project in Cursor.

If you maintain a global Cursor rule (Settings → Rules for AI), you can strip the
"Plan Execution", "Quality Assurance", and "Code Style" sections below and keep
only the "Project Context" and "Project-Specific Rules" sections.
-->

## Plan Execution & Working Style

**Default workflow**: Step-by-step with discussion.
- Complete one task/step at a time, then **stop and explain** what was done.
- Wait for the user to say "continue", "next", "yes", etc. before proceeding.
- Ask questions, present options, and discuss design choices at each step.
- **Only if user says "build-everything"**: Execute all todos without pausing.

**Explain-before-write**: Before writing any non-trivial function, state:
  1. What it does and what invariants it maintains
  2. What can go wrong (error paths, edge cases)
  3. How it interacts with existing state
  Wait for confirmation before writing the implementation.

**Review cadence**: Implement in small chunks. Explain what was done.
Wait for explicit approval before continuing. No large batch changes.

**Blockers**: If genuine ambiguity arises mid-step, stop immediately and ask.
Do not guess and continue.

**Scope**: Only touch what's directly asked. Flag other issues as numbered
notes (e.g. "Note [1]: X could be improved") — never fix them silently.

**Change surface**: Before any edit touching more than one function, list
exactly which types and functions will change and why.

**Assumptions**: Never silently resolve domain ambiguity (units, types,
ownership, conventions). Always surface and confirm.

**Verification**: When uncertain about existing code, types, schemas, or
project structure — use available tools (MCP, file reads, search) to check
before responding. If tools cannot resolve the ambiguity, ask. Never
reconstruct or guess from memory when ground truth is accessible.

**Rollback**: For non-trivial changes, briefly note what reverting would
require — what to revert, what state needs cleaning up.

**Cross-referencing**: When implementing patterns from other repos, show the
reference code and confirm the approach before writing.

**Git commits**: **NEVER run `git commit` automatically.** When work is complete:
1. Remind the user to commit
2. Suggest a commit message following conventional commits format
3. Wait for the user to execute the commit manually

Do not stage files or commit without explicit user request.

---

## Quality Assurance (Rust)

**Before any commit or PR:**
1. Run `cargo check` - must pass
2. Run `cargo clippy -- -D warnings` - must pass with zero warnings
3. Run `cargo test` - all tests must pass
4. Run `cargo fmt --check` - code must be formatted

Or simply: `just ci`

**During development:**
- Fix clippy warnings immediately, don't accumulate them
- Add tests for new functionality
- Update tests when changing behavior

**If clippy fails:**
- Fix the warnings, don't suppress them with `#[allow(...)]` unless absolutely necessary
- If suppression is needed, add a comment explaining why

---

## Code Style (Rust)

- **Idiomatic Rust**: Prefer standard traits (`From`, `TryFrom`, `Display`,
  `Default`, etc.) over custom methods. Use `_` prefix for intentionally
  unused fields instead of `#[allow(dead_code)]`.
- **Module docs** (`//!`): Keep these.
- **Inline/function comments**: Minimal. Only comment non-obvious logic,
  important context, or *why* — not *what*.
- **Doc comments** (`///`): Only add when they provide genuine value beyond
  what the type signature and name already communicate. Assume the reader is
  an experienced Rust developer. Never write doc comments that restate the
  function name, describe obvious behaviour, or explain standard Rust patterns.
  A missing doc comment is better than a pointless one.
- **No panics**: No `unwrap()`, `expect()`, or `panic!()` outside tests.
  Use `unwrap_or`, `unwrap_or_else`, `?`, or proper error handling.
- **Error handling**: `thiserror` for library errors, `anyhow` for
  application/CLI errors. Prefer typed errors over string messages.
- **State and concurrency**: Before implementing anything that touches shared
  state, channels, or async, explicitly call out ownership and
  synchronization assumptions for discussion.
- **Dependencies**: Do not introduce new crates without discussing first.
  Prefer what's already in the workspace `Cargo.toml`.
- **Import style**: All `use` statements at the top of scope. Never qualify
  types inline if used more than once. Exceptions: single-use method
  references (`.map(ToString::to_string)`) and derive paths
  (`#[derive(serde::Deserialize)]`).

---

## Project Context

**What this project does:**
Production-grade Rust microservice skeleton. Provides lifecycle management,
health/metrics HTTP endpoints, OpenTelemetry tracing, structured logging, and
Docker support. Rename the `example-*` crates to match your service domain.

**Architecture:**
- `tokio-graceful-shutdown` backs the `AppManager` trait — subsystems are async
  functions registered with a `SubsystemHandle`
- Health endpoint (`:8080/health`) returns 200 when `RunningStatus` is ready,
  503 otherwise
- Metrics endpoint (`:9090/metrics`) exposes Prometheus exposition format,
  including system and Tokio runtime metrics

**Workspace crates:**
- `crates/app/` — Binary: CLI (`clap`), bootstrap wiring, tracing + metrics setup
- `crates/app-core/` — Library: `AppManager` trait, `RunningStatus`, health server
- `crates/example-config/` — Domain config structs (rename to `<service>-config`)
- `crates/example-core/` — Domain error types and shared primitives (rename to `<service>-core`)
- `crates/example-store/` — Storage trait + `mockall` feature-gate pattern (rename to `<service>-store`)
- `crates/example-service/` — Domain logic wired to store via trait abstraction (rename to `<service>-service`)

**Build and run:**
```bash
just build       # cargo build
just run         # cargo run
just ci          # check + clippy + test + fmt
just health      # curl http://localhost:8080/health
just metrics     # curl http://localhost:9090/metrics
```

**Ports:**
- `8080` — Health check (`/health`)
- `9090` — Prometheus metrics (`/metrics`)

**Key workspace dependencies (do not duplicate in crate Cargo.toml):**
- `tokio`, `actix-web`, `clap`, `serde`, `serde_yaml`
- `thiserror`, `tracing`, `tracing-subscriber`, `tracing-opentelemetry`
- `metrics`, `metrics-exporter-prometheus`, `sysinfo`, `tokio-metrics`
- `opentelemetry`, `opentelemetry_sdk`, `opentelemetry-otlp`
- `tokio-graceful-shutdown`, `mockall`

**Toolchain:** Rust 1.85, Edition 2024 (pinned via `rust-toolchain.toml`)

---

## Project-Specific Rules

- **Renaming domain crates**: When renaming `example-*` crates, update all
  `[workspace.dependencies]` entries in `Cargo.toml` and all internal
  `path = "crates/..."` references before touching any source.
- **Adding subsystems**: Subsystems must be registered with `SubsystemHandle`
  in `crates/app/src/lib.rs`. Discuss lifecycle ordering before adding.
- **Workspace lints are enforced**: `unwrap_used`, `expect_used`, `panic`,
  `unsafe_code` are all denied at the workspace level. Do not add
  `#[allow(...)]` to silence them without discussion.
- **Metrics naming**: Follow the existing `system_*` and `tokio_*` prefix
  conventions. Discuss new metric names before adding.
- **Config changes**: Any new config fields must be added to the relevant
  `example-config` struct and reflected in `docker/` config files if applicable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qmilangowin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
