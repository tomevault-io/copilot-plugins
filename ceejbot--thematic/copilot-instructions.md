## thematic

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**Thematic** (v0.1.3) is a bidirectional converter between VSCode and Zed editor theme formats. It converts color themes, icon themes, and extension manifests. Works on macOS, Linux, and Windows.

## Build / Lint / Test

There's a `.justfile` (leading dot!) with convenience recipes.

### Preferred commands
- **Test:** `cargo nextest run` (project preference over `cargo test`)
- **Lint:** `cargo clippy --all-targets`
- **Format:** `cargo +nightly fmt --all` (nightly required; config in `.rustfmt.toml`)
- **CI check:** `just ci` (runs tests, clippy, and fmt --check)
- **Quick lint+fix:** `just lint` (clippy --fix + fmt)
- **Single test:** `cargo nextest run <test_name>`
- **Run binary:** `cargo run -- <args>` (e.g., `cargo run -- vz catppuccin`)

### Testing notes
- Tests prefixed `no_ci_` require locally-installed editor extensions and will fail in clean environments.
- All other tests use fixtures in `fixtures/` and should always pass.

## Code Style

- **Formatting:** Nightly rustfmt with `.rustfmt.toml` config (max_width=120, style_edition=2024, module-level imports grouped std/external/crate).
- **Lints:** `deny(clippy::unwrap_used)` outside tests, `deny(unsafe_code)` everywhere (set in `src/lib.rs`).
- **Error handling:** Use `Result<T, ThemeError>` with the crate's error types in `src/errors.rs` (derived via `thiserror`). No `unwrap()` outside tests.
- **Naming:** Standard Rust conventions (snake_case functions/variables, CamelCase types).
- **Doc comments:** `//!` for modules, `///` for items.

## Architecture

### CLI (`src/main.rs`)
Binary uses `clap` with four subcommands:
- `vscode-to-zed` / `vz` — Convert VSCode themes to Zed
- `zed-to-vscode` / `zv` — Convert Zed themes to VSCode
- `zed-list` / `zed` — Search installed Zed themes
- `vs-code-list` / `vsc` — Search installed VSCode themes

Global flags: `-q` (quiet), `-v` (verbose/debug), `--dry-run` (preview without writing).

When multiple candidates match a name, the best match is selected via `strsim::normalized_levenshtein`.

### Module layout
```
src/
├── main.rs              # CLI entry point and orchestration
├── lib.rs               # Crate root; re-exports + lint attrs
├── errors.rs            # ThemeError enum (thiserror)
├── grouping.rs          # Theme family grouping by name similarity
├── icon_files.rs        # IconFileManager for copying icon assets
├── conversion_tests.rs  # Integration tests (cfg(test) only)
├── editors/
│   ├── mod.rs           # Extension + ThemeFile traits, globdir()
│   ├── vscode/          # VsCodeExtension, VsCodeTheme, VsCodePackageJson, VsCodeIconTheme
│   └── zed/             # ZedExtension, ZedTheme, ZedManifest, ZedIconTheme, InstalledExtensions
└── convert/
    ├── mod.rs           # Convert module interface
    ├── code_to_zed.rs   # VSCode → Zed (From<&VsCodeTheme> for ZedTheme)
    └── zed_to_code.rs   # Zed → VSCode (From<&ZedTheme> for VsCodeTheme)
```

### Key traits
- **`Extension`** (in `editors/mod.rs`) — Requires `Sized`. Defines `search()`, `read()`, `write()`, `write_to()`, `extensions_path()`, `name()`, `manifest()`, `themes()`, `icon_themes()`, `official_path()`. Implemented by `VsCodeExtension` and `ZedExtension`.
- **`ThemeFile`** — `read()`, `write_to()`, `from_bytes()`. Implemented by theme family types.

### Key implementation details
- `pub use editors::*` in lib.rs is required — internal code uses `crate::vscode::` and `crate::zed::` paths through the wildcard re-export chain.
- `globdir()` in `editors/mod.rs` is `pub(crate)` — uses `fast-glob` + `walkdir` for filesystem pattern matching.
- Theme family grouping (`src/grouping.rs`) uses hybrid prefix + variant clustering to group related themes (e.g., "Rosé Pine", "Rosé Pine Moon", "Rosé Pine Dawn" into one family).
- `InstalledExtensions` (`editors/zed/installed.rs`) uses `#[serde(flatten)]` for forward-compatible handling of unknown extension types in Zed's `index.json`.
- `ZedThemeStyle` has ~144 optional `String` fields in a flat struct (mirrors Zed's flat JSON schema). Fields are grouped with section comments.
- Platform-specific paths: Zed uses `#[cfg(target_os)]` for macOS/Linux/Windows; VSCode uses `~/.vscode/extensions` everywhere.
- Both `write_to()` methods log a warning before overwriting an existing extension directory.

### Conversion flow
1. `Extension::search()` finds candidate extensions by glob-matching the user's input
2. Best match selected (Levenshtein distance if multiple candidates)
3. `From` impl converts the extension type (e.g., `ZedExtension::from(vscode_ext)`)
4. Internally, individual themes are converted via `From<&VsCodeTheme> for ZedTheme` (or reverse)
5. `Extension::write()` writes theme files, icon themes (with file copying), and manifest to the target editor's extension directory

## Reference

### Fixtures and schemas
- `fixtures/vscode/` — Catppuccin and Rosé Pine VSCode extension fixtures
- `fixtures/zed/` — Catppuccin, Rosé Pine, and Serendipity Zed extension fixtures (including icon themes)
- `docs/schemas/` — JSON schemas for both VSCode and Zed theme formats

### Dependencies (notable)
`clap` (CLI), `serde`/`serde_json`/`toml` (serialization), `thiserror` (errors), `fast-glob`+`walkdir` (filesystem), `strsim` (fuzzy matching), `slug` (name normalization), `owo-colors` (terminal output), `home` (home directory), `lovely_env_logger` (logging)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ceejbot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ceejbot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
