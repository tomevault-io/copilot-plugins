## crossfs

> `crossfs` is a Rust library that provides a synchronous `std::fs`-style file API that works identically on native and WASM.

# crossfs

## Summary

`crossfs` is a Rust library that provides a synchronous `std::fs`-style file API that works identically on native and WASM.

## Project Type & Toolchain

- **Language:** Rust, edition 2024
- **Rust toolchain:** stable (`rustc 1.94.0+`); no `rust-toolchain.toml`
- **Targets:** native (x86_64/aarch64) and `wasm32-unknown-unknown`
- **JS tooling:** `wasm-pack 0.15.0`, `trunk` (Wasm bundler), `bun` (package manager for dev-dependencies)

## Build & Validate Commands

Always run from the repository root (`/home/vivim/crossfs`) unless noted.

```bash
# Build the library (native)
cargo build

# Run all native tests (fast, under 1s)
bun run test:native

# Lint (clippy)
bun run lint

# Format (rustfmt)
bun run format

# Benchmark the library
bun run bench

# Web tests
bun run test:web
```

## Validation Checklist

Before submitting changes, always verify:

1. `bun run format` — no formatting issues
2. `bun run lint` — no clippy warnings
3. `bun run test:native` — all native tests pass
4. `bun run test:web` — all web tests pass

---
> Source: [evilbocchi/crossfs](https://github.com/evilbocchi/crossfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
