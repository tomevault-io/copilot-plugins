## timestretch-rs

> The following checks run on every push to `main` and on every pull request. All must pass.

# CLAUDE.md

## CI Checks

The following checks run on every push to `main` and on every pull request. All must pass.

### Tests
```bash
cargo test --all-targets
```
Runs on Ubuntu, macOS, and Windows with stable Rust, plus Ubuntu with MSRV (1.82.0).

### Clippy
```bash
cargo clippy --all-targets -- -D warnings
```
All warnings are treated as errors.

### Format
```bash
cargo fmt --all --check
```
Code must be formatted with `rustfmt`.

### Documentation
```bash
RUSTDOCFLAGS="-D warnings" cargo doc --no-deps
```
Documentation must build without warnings.

---
> Source: [robmorgan/timestretch-rs](https://github.com/robmorgan/timestretch-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
