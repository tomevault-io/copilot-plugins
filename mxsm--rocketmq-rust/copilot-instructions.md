## rocketmq-rust

> This is the standalone Rust backend for the Tauri app.

## Claude Code

This is the standalone Rust backend for the Tauri app.

Do not rely on root workspace validation for this directory.

## Mandatory validation

Run from this directory after every Rust code change:

```bash
cargo fmt --all
cargo clippy --all-targets --all-features -- -D warnings
```

## Test policy

- Run only the affected tests by default.
- Do not run the full test suite unless the change is broad or shared.

## Examples

```bash
cargo test some_test_name
cargo test --lib
cargo test
```

## Cross-project rule

- If shared Rust crates referenced by this project are modified, validate those shared crates from the repository root as needed.

---
> Source: [mxsm/rocketmq-rust](https://github.com/mxsm/rocketmq-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
