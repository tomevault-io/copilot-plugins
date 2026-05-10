## query-system-info

> This is a **Rust workspace** containing a cross-platform system information library with three crates:

# AGENTS.md

## Cursor Cloud specific instructions

This is a **Rust workspace** containing a cross-platform system information library with three crates:

| Crate | Path | Description |
|---|---|---|
| `query_system_info` | `/workspace` | Core Rust library + CLI binary |
| `js-abi` | `/workspace/js-abi` | Node.js NAPI-RS bindings |
| `py-abi` | `/workspace/py-abi` | Python PyO3/maturin bindings |

### Key commands

Standard build/test/lint commands are in `Makefile`. Quick reference:

- **Check**: `cargo check --workspace`
- **Build**: `cargo build --workspace` (or `make build` for release)
- **Test**: `cargo test --workspace` (15 unit tests across all 3 crates; 2 doc-tests in `src/lib.rs` have pre-existing compilation errors)
- **Lint**: `cargo clippy --workspace` (warnings only, no errors)
- **Format check**: `cargo fmt --check`
- **Run CLI**: `cargo run`
- **Run example**: `cargo run --example basic_usage`

### JS bindings

- `cd js-abi && yarn install && yarn build` produces `dist/index.node`
- Test with `node examples/basic_usage.js`

### Python bindings

- Requires `python3-dev` and `python3.12-venv` system packages
- Create a venv: `python3 -m venv .venv && source .venv/bin/activate`
- Build: `maturin build --manifest-path py-abi/Cargo.toml`
- Install: `pip install --no-deps target/wheels/py_query_system_info-*.whl`
- The `pyproject.toml` lists `query_system_info` as a pip dependency which doesn't exist on PyPI; always use `--no-deps` when installing the wheel, or use `maturin develop` only after removing that dependency from pyproject.toml

### Caveats

- The `js-abi` and `py-abi` sub-crates use `edition = "2024"`, which requires Rust 1.85+. Run `rustup update stable` if the default toolchain is older.
- After `rustup update`, you may need to run `hash -r` in bash so the shell picks up the new `cargo`/`rustc` binaries.
- The Python example (`examples/basic_usage.py`) may panic on `unwrap()` in `get_connection_by_state` — this is a pre-existing bug in `py-abi/src/lib.rs`.
- No external services, databases, or Docker are required. The library reads directly from OS interfaces (`/proc` filesystem on Linux).

---
> Source: [wuyouMaster/query_system_info](https://github.com/wuyouMaster/query_system_info) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
