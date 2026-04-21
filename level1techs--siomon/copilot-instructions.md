## siomon

> siomon is a Linux hardware information and real-time sensor monitoring tool

# siomon - Agent Context

## Project Overview

siomon is a Linux hardware information and real-time sensor monitoring tool
written in Rust. The binary is called `sio`, the package is called `siomon`.
Licensed under MIT.

Repository: https://github.com/level1techs/siomon

Two operating modes:
- **TUI dashboard** -- real-time sensor monitoring (default when stdout is a
  terminal and no subcommand is given).
- **One-shot info** -- hardware details via subcommands or a sensor snapshot.

## Key Sources

- `Cargo.toml` -- edition, rust-version, feature flags, build profile, and
  dependencies.
- `src/` -- source tree organized by concern. Each top-level module has a
  doc comment or `mod.rs` explaining its role.
- `src/cli/` -- CLI subcommands and flags.
- `src/config/` -- config file loading.
- `kmod/sinfo_io/` -- DKMS-based Linux kernel module.
- `.github/workflows/` -- CI, release, and packaging workflows.
- `packaging/aur/AGENTS.md` and `packaging/launchpad/AGENTS.md` -- packaging
  context and pointers to full documentation.

## Build and Test

```bash
cargo test --all-features
cargo clippy --all-features -- -D warnings -A dead_code
cargo build --release --all-features
```

## Formatting

Always run `cargo fmt` on any Rust source files you modify. CI enforces `cargo fmt --check` and will fail if formatting is off.

---
> Source: [level1techs/siomon](https://github.com/level1techs/siomon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
