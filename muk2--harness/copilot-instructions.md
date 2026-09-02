## harness

> Root instructions for coding agents working in this repository. Per-crate `CLAUDE.md`

# CLAUDE.md — agent instructions for the harness repo

Root instructions for coding agents working in this repository. Per-crate `CLAUDE.md`
files may add local rules; this is the map and the non-negotiables.

## What this is

A self-hosted per-user agentic harness: a Rust daemon (`harnessd`) plus thin TUI and
iOS clients that connect over Tailscale. One daemon = one user. The full design is
[docs/DESIGN.md](docs/DESIGN.md); read it before large changes.

## Repo map

- `crates/harness-proto` — the wire protocol (serde types) + golden-fixture tests. **The
  contract.** Mirrored in Swift under `ios/`.
- `crates/harness-core` — provider layer (`provider/`), SQLite event log (`store.rs`),
  agent runtime (`runtime.rs`), config (`config.rs`). No transport code.
- `crates/harnessd` — axum WS+REST server (`ws.rs`, `rest.rs`, `server.rs`) + CLI
  (`cli.rs`). A thin shell over `harness-core`.
- `crates/harness-tui` — ratatui client. `app.rs` holds pure, testable update logic;
  `ui.rs` is view-only; `client.rs` owns the socket. `--script` mode is headless.
- `crates/harness-relay` — stateless, content-free APNs relay.
- `ios/` — SwiftUI client, developed natively on the Mac (not containerizable).

## The rules

1. **Protocol-first.** Any client-visible change starts in `harness-proto` **and**
   `proto-fixtures/`. A protocol change that doesn't update a fixture is a bug — the
   fixture contract tests (Rust `crates/harness-proto/tests/fixtures.rs` and Swift
   `ios/HarnessAppTests/FixtureTests.swift`) exist to fail loudly on drift. Clients must
   ignore unknown event/message types (`#[serde(other)]` / `.unknown`), never error.
2. **No `unwrap()`/`expect()` on daemon request paths.** Return `CoreError` and let the
   REST/WS layer map it. Panics are only acceptable in `init`-time setup and tests.
3. **Tracing spans on all handlers.** Structured fields, never secrets. The event log
   and logs both store references (`provider: anthropic`), never key values.
4. **Vertically slice work.** "Tool approval end-to-end" = proto event + runtime gate +
   TUI card in one change, not a layer at a time. Keep the fixture and both sides in sync.
5. **Keep backends behind the trait.** Every provider is independently disable-able via
   config (`enabled = false`). Never let vendor-specific behavior leak past
   `harness-core::provider`.
6. **Safety is structural.** Presets are allowlist-first (`default = "forbid"`). Never
   add an implicit path that lets the model reach a tool the preset didn't declare.

## Build / test / verify

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings   # CI runs with -D warnings
cargo test --workspace
bash scripts/e2e-smoke.sh          # boots the daemon + drives the TUI headless
```

`harness-tui --script "<msg>"` is the self-verification harness: it drives a full turn
and prints the event stream. Use it (and the mock provider) to check runtime changes
without an API key.

## Where milestones plug in

Search the source for `M2` / `M3` / `M4` markers — they mark the exact seams for tools +
approvals (M2), more providers + presets (M3), and pairing + operability (M4). The types
already exist (`Event::Tool*`, `ApprovalClass`, preset TOML); the work is wiring them.

## Conventions

- Conventional Commits (`feat:`, `fix:`, `ci:`, `docs:`, `refactor:`, `chore:`, `test:`).
- Branch from `develop`; `main` is always deployable. PRs small and single-concern.
- New protocol event? → type in `harness-proto`, `type_tag()` arm, a fixture, Swift
  mirror, and handling in `harness-tui/app.rs`. All five, same change.

---
> Source: [muk2/harness](https://github.com/muk2/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
