## advisor

> The repo on `main` is **Rust-only** (`advisor-rs` / `Cargo.toml`). There is no

# Advisor — Agent Instructions

## Cursor Cloud specific instructions

### Environment install

The repo on `main` is **Rust-only** (`advisor-rs` / `Cargo.toml`). There is no
`pyproject.toml`. Use:

```bash
bash scripts/setup.sh
```

Cursor Cloud reads `.cursor/environment.json`, which runs that script before the
agent starts. The script is idempotent (safe to re-run).

### Quick reference

- **Setup / sync deps**: `bash scripts/setup.sh` (or `cargo build --locked` after toolchain is ready)
- **Toolchain**: `rust-toolchain.toml` pins `stable` (needs edition-2024 support — Rust 1.85+)
- **Build**: `cargo build`
- **Tests**: `cargo test --locked`
- **Lint**: `cargo clippy --all-targets -- -D warnings`
- **Format**: `cargo fmt --check`
- **All checks**: `make check` (clippy + fmt + test)
- **Run CLI**: `cargo run -- <subcommand>` (e.g. `version`, `plan`, `presets`, `ui`)
- **Web dashboard**: `cargo run -- ui --port 8765`

### Gotchas

- **No Python package on `main`**: `tests/*.py` and `uv sync` are legacy; CI uses `cargo test --locked`.
- **Zero runtime deps** for the binary — only the Rust toolchain is required to build.
- `Makefile` targets wrap `cargo` (see `make help` via targets in the Makefile).
- The `claude` and `codex` CLIs are not needed for development/testing — only for the live pipeline.
- If `cargo build` fails with `edition2024` / `idna_adapter`, run `rustup update stable` (handled by `scripts/setup.sh`).

---
> Source: [vzwjustin/advisor](https://github.com/vzwjustin/advisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
