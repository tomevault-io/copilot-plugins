## tracing-scribe

> Guidance for Claude Code when working in this repository.

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

`tracing-scribe` is a Rust library providing `ConsoleLayer`, a
`tracing-subscriber` layer that formats spans and events as a hierarchical,
colourful terminal tree with status icons, automatic span timings, and error
propagation.

## Layout

- `src/lib.rs` — single-file implementation plus embedded `#[cfg(test)]` tests
- `examples/full.rs`, `examples/custom_colors.rs` — runnable demos
- `.github/workflows/ci.yml` — build / test / clippy / fmt on push and PR

## Common commands

```bash
cargo build --all-targets
cargo test
cargo clippy --all-targets -- -D warnings
cargo fmt --all -- --check
cargo run --example full
cargo run --example custom_colors
```

## Notes

- Rust edition 2024.
- Dependencies: `tracing`, `tracing-subscriber`, `owo-colors`.
- Tests use a `MockWriter` that captures output, strip ANSI escapes and
  durations, then snapshot with `insta`. When test output changes
  intentionally, update the inline snapshots with `cargo insta review`.

---
> Source: [helsing-ai/tracing-scribe](https://github.com/helsing-ai/tracing-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
