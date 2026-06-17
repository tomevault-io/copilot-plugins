## dcr

> cargo build --release && ./target/release/dcr setup

# AGENTS.md

## Quick start

```bash
cargo build --release && ./target/release/dcr setup
```

## Pre-PR (CI) gate

```bash
cargo fmt --all --check
cargo clippy --all-targets --all-features -- -D warnings
cargo check --all-targets --all-features
cargo test
```

CI (`.github/workflows/ci.yml`) runs fmt, clippy, check — **not** tests.

## Tests

- Integration tests in `tests/` build the dcr binary and run it in temp dirs under `sandbox/cli-tests/` (gitignored).
- Some tests **skip** if no C compiler (`gcc`/`clang`/`cc`) is found at runtime.
- Compiler can be overridden via `DCR_COMPILER` or `DCR_CC` env vars.
- Tests use `$CARGO_BIN_EXE_dcr` when available, else `target/debug/dcr`.

## Architecture

- **Single Rust package** (edition 2024), not a workspace. Only one binary: `dcr`.
- **Entrypoint**: `src/main.rs` — matches `args[1]` against command names.
- **Compiler backends** in `src/core/builder/`: `unix_cc` (gcc/clang), `msvc` (cl/clang-cl), `gas`, `nasm`. Dispatched by matching compiler name string.
- **Config**: `dcr.toml` (per-project), `~/.dcr/config.toml` (registry), `~/.dcr/index.json` (index). Defaults: `language=c`, `standard=c11`, `compiler=clang`, `kind=bin`.
- **`build.rs`** embeds `TARGET` as `DCR_TARGET` env var (shown in `--version`).
- **CLI output** must be in English. Use `utils::log` for messaging.

## Key commands

| Command | Notes |
|---|---|
| `dcr new <name>` | Creates project dir |
| `dcr init` | Init from CWD (must be empty) |
| `dcr build [--debug\|--release] [--target <triple>]` | Default profile: `--debug` |
| `dcr run [--debug\|--release]` | Only for `kind = "bin"` |
| `dcr clean [--debug\|--release] [--all]` | `--all` for workspace roots |
| `dcr tree` | Dependency tree viewer |
| `dcr test [--init]` | Runs/test project tests |
| `dcr gen <vscode\|clion\|compile-commands>` | IDE integration |
| `dcr --update` | Self-update from GitHub Releases |

## Cross-compilation

`--target` accepts short names (`linux`, `macos`, `windows`) or full triples. In CI, Linux cross-builds use `cargo-zigbuild`. When `build.target` is set in `dcr.toml`, DCR auto-injects `--target=<value>` into cflags (clang). Bare-metal targets (containing `none`/`-elf`/`eabi`/`baremetal`) skip DCR's default compiler flags.

## Dependencies (Cargo)

reqwest (blocking+json+rustls-tls), serde, toml, git2, openssl (vendored), self-replace, sha2, ctrlc.

---
> Source: [dexoron/dcr](https://github.com/dexoron/dcr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
