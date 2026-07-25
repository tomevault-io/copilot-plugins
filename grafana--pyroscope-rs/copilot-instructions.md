## pyroscope-rs

> Before committing, always run:

# pyroscope-rs

# Pre-Commit Checks

Before committing, always run:

```bash
cargo fmt --all
cargo clippy --all-targets --all-features -- -D warnings
cargo test
```

All must pass with no errors before creating a commit.

---
> Source: [grafana/pyroscope-rs](https://github.com/grafana/pyroscope-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
