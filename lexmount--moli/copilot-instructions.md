## moli

> Before committing changes that modify Rust source code or Rust build metadata

Before committing changes that modify Rust source code or Rust build metadata
(such as `Cargo.toml`, `Cargo.lock`, or `rust-toolchain`), run all of the
following from the repository root and ensure they pass:

```sh
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo nextest run --no-fail-fast
```

These commands are not required when the change set contains no Rust source or
Rust build metadata changes.

---
> Source: [lexmount/moli](https://github.com/lexmount/moli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
