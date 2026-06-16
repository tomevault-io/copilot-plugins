## tiri

> Es necesaria la paridad en funcionamiento con i3/sway

# Repository Guidelines

Es necesaria la paridad en funcionamiento con i3/sway
Hay que evitar todo lo posible dar palos de ciego.
## Project Structure & Module Organization
- `src/`: compositor + layout core (tiling, input, rendering). Layout logic lives under `src/layout/`.
- `tiri-config/`, `tiri-ipc/`: config parsing and IPC crates used by the compositor.
- `docs/`, `resources/`: documentation and static assets.
- `tiri-visual-tests/`: visual/snapshot test helpers.
- `config.kdl`: example config; user config typically lives at `~/.config/tiri/config.kdl`.

## Build, Test, and Development Commands
- `cargo build`: compile the compositor and supporting crates.
- `cargo test`: run Rust unit/integration tests.
- `cargo test -q move_`: quick filter for movement/layout tests.
- `cargo insta test` / `cargo insta review`: run and update snapshot tests (used in layout/visual checks).
- `cargo fmt` / `cargo clippy`: format and lint (use `rustfmt.toml` and `clippy.toml`).
- Optional (Nix): `nix develop` to enter a dev shell if you use `flake.nix`.

## Coding Style & Naming Conventions
- Rust, 4-space indentation, `rustfmt` enforced.
- Modules/types in `CamelCase`, functions/vars in `snake_case`.
- Prefer explicit, small helpers for layout logic; avoid duplicated tree traversal logic.
- Keep ASCII in source unless the file already uses Unicode.

## Testing Guidelines
- Core layout tests live in `src/layout/tests.rs` and `src/layout/tests/*`.
- Snapshot tests use `insta` (run `cargo insta test`, update with `cargo insta review`).
- Add targeted tests for i3-like behavior changes (movement, container flattening, tabbed/stacked).

## Commit & Pull Request Guidelines
- Use short, imperative commit subjects (e.g., “Fix tab bar anchoring”).
- Include scope when helpful (e.g., `layout:` or `input:`).
- PRs should include: summary, tests run, and screenshots/GIFs for visual/UI changes.
- Link related issues or roadmap sections when behavior changes affect i3/sway parity.

## Configuration & Security Notes
- Keep `config.kdl` as a working example; avoid breaking fields used in `tiri-config`.
- Honor `block_out_from` rules when rendering text in bars/indicators for screencast safety.

---
> Source: [pablocpas/tiri](https://github.com/pablocpas/tiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
