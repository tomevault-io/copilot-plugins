## sourcefour

> Use the justfile. `just --list` shows everything, grouped: run, build, check, dist.

# Sourcefour — agent notes

## Commands

Use the justfile. `just --list` shows everything, grouped: run, build, check, dist.

- `just run [PATH]` / `just demo` — launch the app
- `just test [NAME-FILTER]`, `just test-crate <crate>` — tests
- `just check` — the full pre-push gate, same as CI

Drop to `cargo nextest run …` directly when you need flags no recipe covers
(`-E` filter expressions, `--no-capture`). Never `cargo test` — nextest is the
runner here, and it does not run doctests. We don't write doctests; put
examples in `#[test]` functions instead.

---
> Source: [HelgeSverre/sourcefour](https://github.com/HelgeSverre/sourcefour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
