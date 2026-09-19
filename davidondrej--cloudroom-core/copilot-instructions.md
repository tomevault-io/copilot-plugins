## cloudroom-core

> - Keep one Rust package and one service; app integrations call the API.

# Cloudroom core

- Keep one Rust package and one service; app integrations call the API.
- Run `cargo fmt --check`, `cargo clippy --locked --all-targets -- -D warnings`, and `cargo test --locked --all-targets`.
- Use disposable databases for tests. Only the owner applies production migrations.
- SQL and application instructions live in `docs/database/`.
- Keep credentials and internal notes in ignored files. Review changes before publication.

---
> Source: [davidondrej/cloudroom-core](https://github.com/davidondrej/cloudroom-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
