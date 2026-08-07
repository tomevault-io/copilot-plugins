## wiff

> wiff is a terminal-centric diff and code-review utility: it captures a diff,

## Project Overview

wiff is a terminal-centric diff and code-review utility: it captures a diff,
lets you browse and annotate it with syntax highlighting, and persists the
review as a local session that both a human (via the TUI) and an agent (via CLI
and a skill) can read and write concurrently. The binary is `wiff`.

The design reference is `docs/spec.md`.

## Build Commands

```bash
make check        # Quick syntax check (cargo check)
make build        # Build all binaries
make lint         # Clippy, with warnings promoted to errors
make test         # Full test suite (cargo nextest run)
make fmt          # Format all code (cargo +nightly fmt)
```

 - to rustfmt directly use `cargo +nightly fmt`
 - to run tests directly use `cargo nextest run` as it is faster than cargo test.
 - `make lint` promotes clippy warnings to errors. Every checkpoint must be
   clippy-clean: never leave or ignore a warning, even a pre-existing one. If a
   warning is genuinely not worth fixing, suppress it deliberately with a
   justified `#[allow(...)]` rather than letting it ride.

## Code Structure

 - Rust crates are found in the `crates` directory, and are linked into the
   workspace members list.
 - Cargo.toml workspace.members must be kept in alphabetical order
 - workspace dependencies are used throughout.
 - Cargo.toml dependencies are always kept in alphabetical order
 - Prefer to use the `anyhow.workspace = true` form when adding a dependency to
   a crate

## Coding Conventions

 - Always preserve existing comments when modifying code; both doc comments and
   inline comments.
 - Except in tests, avoid `.unwrap()` and `panic!`. Prefer to propagate errors
   using the `?` operator when in a function that returns a `Result`. If a panic
   is unavoidable, use `.expect("REASON WHY")` instead of a bare `.unwrap()`.
 - In tests, always assert on the full rendered output a human would see -- never
   examine a substring or a single field in isolation. Do not use
   `str.contains("something")` or other "keyhole" checks; examine the full value.
   Prefer `wince::snapshot!` or `wince::assert_eq!` for value comparisons.
 - If a test has unstable/variable output (eg: temporary file paths), preprocess
   the string to replace the known temporary path with a constant like TMPDIR
   before asserting.
 - When adding `use` imports, place them in a block at the top of the file (or
   at the top of the `mod`), organized by the formatter.

---
> Source: [wez/wiff](https://github.com/wez/wiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
