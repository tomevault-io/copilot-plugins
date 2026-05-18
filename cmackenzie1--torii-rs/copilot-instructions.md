## sqlx

> When writing rust sqlx queries

- Never use the sqlx query macros `query!` or `query_as!` when writing sqlx queries
- Always use the non-macro version when writing sqlx queries
- Prefer passing borrowed values when using bind

---
> Source: [cmackenzie1/torii-rs](https://github.com/cmackenzie1/torii-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
