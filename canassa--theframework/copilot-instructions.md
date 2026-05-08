## theframework

> A fast Python HTTP framework powered by io_uring and greenlets. No async/await—write synchronous code with the runtime handling concurrency.

# theframework

A fast Python HTTP framework powered by io_uring and greenlets. No async/await—write synchronous code with the runtime handling concurrency.

## Folder Structure

- `theframework/` — Python package with main framework code
- `src/` — Zig source for io_uring integration
- `stubs/` — Type stubs for C extensions
- `tests/` — Pytest tests
- `examples/` — Example applications
- `plans/` — Design and implementation documentation
- `build.zig` — Zig build script

## Commands

- `zig build test` — Run Zig tests
- `zig build` — Build the extension (required before pytest)
- `uv run pytest tests/` — Run Python tests
- `uv run mypy` — Run type checker
- `uv run ruff check` — Run linter

## Rules

- ALWAYS run all tests and type checker after making changes

## Key Implementation Notes

- Custom HTTP server in Zig
- Greenlets handle concurrency (no async/await)
- io_uring for high-performance I/O
- Pre-fork parallelism for multi-core scaling

---
> Source: [canassa/theframework](https://github.com/canassa/theframework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
