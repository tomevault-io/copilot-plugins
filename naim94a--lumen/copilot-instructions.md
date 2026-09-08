## lumen

> This is a rust project. It exposes TCP services on user configured ports, and optional TLS.

This is a rust project. It exposes TCP services on user configured ports, and optional TLS.

## Agent Instructions
- keep changes minimal
- write tests when appropriate
- changes must pass `cargo check`, `cargo clippy` & `cargo test`
- do not add dependencies, commit or push without explicit approval
- preserve existing protocol behaviour unless the task explicitly requires changing it.

## Structure
- `lumen` - main binary logic for lumina's http api & binary server.
- `common`
  - `migrations` - database migrations
  - `db` - database helpers
  - `rpc` - contains `serde` implementations for the lumina protocol
  - `web` - a web-api instead of a binary protocol, and more server admin endpoints.

---
> Source: [naim94a/lumen](https://github.com/naim94a/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
