## rusttest

> Rust test project -- a minimal Rust binary crate that prints "Hello, world!".

# CLAUDE.md

## Project Overview

Rust test project -- a minimal Rust binary crate that prints "Hello, world!".

- **Language**: Rust (edition 2021)
- **Crate name**: `rusttest`
- **Type**: Binary (`src/main.rs`)
- **License**: MIT OR Apache-2.0
- **Repository**: https://github.com/AndriyKalashnykov/rusttest

## Build & Test Commands

```bash
cargo build              # compile
cargo test               # run tests
cargo fmt --all -- --check  # check formatting
cargo clippy --all-targets --all-features --workspace -- -D warnings  # lint
cargo doc --no-deps --document-private-items --all-features --workspace --examples  # build docs
cargo publish --dry-run  # verify publishable
cargo audit              # security audit of dependencies
cargo tarpaulin --all-features --workspace --ignore-tests --out lcov  # code coverage
```

## Project Structure

```
src/main.rs          # entry point
Cargo.toml           # package manifest (no dependencies)
Cargo.lock           # dependency lock file
```

## CI/CD

- **CI** (`ci.yml`): test, rustfmt, clippy, docs, publish dry-run, coverage (on push to main and PRs)
- **CD** (`cd.yml`): publish to crates.io on version tags
- **Audit** (`audit.yml`): daily security audit + on Cargo.toml/Cargo.lock changes
- **Dependabot**: weekly updates for cargo and github-actions

## Code Style

- Follow standard `rustfmt` formatting
- Zero `clippy` warnings (`-D warnings`)
- Documentation must compile without warnings (`RUSTDOCFLAGS=-D warnings`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndriyKalashnykov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
