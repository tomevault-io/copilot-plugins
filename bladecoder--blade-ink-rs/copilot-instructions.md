## blade-ink-rs

> - This is a Rust workspace with `runtime`, `compiler`, `rinklecate`, and `conformance-tests`; use the toolchain pinned in `rust-toolchain.toml`.

# Repository Guidelines

- This is a Rust workspace with `runtime`, `compiler`, `rinklecate`, and `conformance-tests`; use the toolchain pinned in `rust-toolchain.toml`.
- Keep changes scoped and add or update focused tests when behavior changes. CI runs `cargo test` and `cargo test -p bladeink --features stream-json-parser`.
- After every task, run `cargo fmt --all` and `cargo clippy --workspace --all-targets --all-features -- -D warnings`. Do not consider the task complete until both pass.

---
> Source: [bladecoder/blade-ink-rs](https://github.com/bladecoder/blade-ink-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
