## herdr-gui

> - Use Cargo for Rust work.

# AGENTS.md

## Build

- Use Cargo for Rust work.
- Use `crepus dev` for local app smoke tests.
- Use `wax`, not `brew`.

## Checks

Run before committing:

```sh
cargo fmt -- --check
cargo clippy --all-targets -- -D warnings
cargo test
```

## Scope

- Keep Herdr as the only backend.
- Treat cmux, Warp, Arc, and Superconductor as UI/product references only.
- Do not add browser panes, plugin UI, marketplace, cloud accounts, or telemetry.
- Prefer small GPUI changes in `crates/herdr-gui/src/main.rs`.
- Prefer Herdr socket wrappers in `crates/herdr-gui/src/herdr.rs`.
- Prefer libghostty changes in `crates/herdr-gui/src/ghostty.rs`.

---
> Source: [undivisible/herdr-gui](https://github.com/undivisible/herdr-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
