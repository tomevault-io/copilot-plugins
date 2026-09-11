## polars-bigquery-client

> - Always use `uv` for running Python commands, managing dependencies, and executing test suites.

# Agent Guidelines

## Python Environment and Package Management

- Always use `uv` for running Python commands, managing dependencies, and executing test suites.
- Do not use global `python`, `pip`, or `pytest` directly. Always prefix with `uv run` (e.g. `uv run pytest`).
- To run unit tests:
  - For `py-arrow-bigquery`: `uv run pytest py-arrow-bigquery`
  - For `py-polars-bigquery`: `uv run pytest py-polars-bigquery`
- For Rust testing: `cargo test`

---
> Source: [pola-rs/polars-bigquery-client](https://github.com/pola-rs/polars-bigquery-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
