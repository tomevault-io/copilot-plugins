## ubiblk

> This repo contains Rust code. Before submitting changes:

# Repository Contribution Guidelines

This repo contains Rust code. Before submitting changes:

1. Format the code:
   ```bash
   cargo fmt --all -- --check
   ```
2. Run the linter:
   ```bash
   cargo clippy --all-targets --all-features -- -D warnings
   ```
3. Run the test suite:
   ```bash
   cargo test
   ```

---
> Source: [ubicloud/ubiblk](https://github.com/ubicloud/ubiblk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
