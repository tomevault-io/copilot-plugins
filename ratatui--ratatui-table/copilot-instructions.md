## ratatui-table

> - This is a single Rust library crate. Keep Table behavior in `src/table.rs` and its existing

# Repository Guidelines

## Project shape

- This is a single Rust library crate. Keep Table behavior in `src/table.rs` and its existing
  concept modules under `src/table/`.
- Preserve the direct crate-root exports of `Table`, `Row`, `Cell`, `TableState`, and
  `HighlightSpacing`.
- Keep the crate usable with `no_std` plus `alloc`; gate standard-library behavior behind `std`.

## Development

- Use `just --list` for local commands.
- Run `just fmt-check`, `just clippy-all`, `just test`, and `just docs` before publication.
- Add deterministic buffer-rendering tests for visible behavior and boundary cases.
- Keep Rustdoc examples runnable and run `just rdme-check` after crate-level documentation changes.

## Changes and releases

- Use Conventional Commits and keep one purpose per change.
- Do not edit generated changelog entries; release-plz owns versions and release notes.
- Treat `.github/`, release configuration, `Cargo.toml`, and `Cargo.lock` as maintainer-owned
  release-sensitive paths.

---
> Source: [ratatui/ratatui-table](https://github.com/ratatui/ratatui-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
