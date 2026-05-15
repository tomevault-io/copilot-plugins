## rust-formatting-rules

> rules for formatting rust code

# Rust Macros

When using `format!` or `assert!` macros in Rust, always put variables inside brackets when possible.
*Example*:

Instead of this:
`format!("Failed to fetch PUT result for {}", filename)`
Do this:
`format!("Failed to fetch PUT result for {filename}")`

---
> Source: [snowflakedb/universal-driver](https://github.com/snowflakedb/universal-driver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
