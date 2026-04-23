## xleak

> Excel terminal viewer written in Rust with TUI, search, formulas, and export capabilities.

# xleak - AI Agent Instructions

Excel terminal viewer written in Rust with TUI, search, formulas, and export capabilities.

**Tech Stack:** Rust 2024, calamine, clap, ratatui + crossterm, anyhow, prettytable-rs, arboard, chrono

**Formats:** `.xlsx`, `.xls`, `.xlsm`, `.xlsb`, `.ods`

**Key files:** `main.rs`, `workbook.rs`, `tui.rs`, `display.rs` in `src/`

## Quick Start

Standard Rust commands: `cargo build`, `cargo test`, `cargo clippy`, `cargo fmt`

**Run with test fixtures:**
```bash
cargo run -- tests/fixtures/test_comprehensive.xlsx -i
```

**Generate test data:**
```bash
source .venv/bin/activate
cd tests/fixtures && python generate_all_tests.py
```

## Code Style

- **Format:** `cargo fmt` (default rustfmt), address all `cargo clippy` warnings
- **Error handling:** Use `anyhow::Result<T>` with `.context()` for error messages
- **Comments:** Minimal, focus on "why" not "what", doc comments for public APIs
- **Pattern matching:** Exhaustive for `CellValue` enum

## Architecture

**Module responsibilities:**
- `main.rs` - CLI parsing, orchestration
- `workbook.rs` - Excel I/O, data extraction (calamine wrapper)
- `tui.rs` - Interactive TUI state and rendering (ratatui)
- `display.rs` - Output formatting (terminal, CSV, JSON, text)

**Key dependencies:**
- `calamine` - Excel parsing, `clap` - CLI framework
- `ratatui` + `crossterm` - TUI, `anyhow` - Error handling
- `prettytable-rs` - Non-interactive display, `arboard` - Clipboard

**Implemented features:** Interactive TUI, search, formulas, clipboard, lazy loading, multi-sheet nav, cell jump, horizontal scrolling

## Error Handling

Use `anyhow` with contextual messages:
```rust
wb.load_sheet(&sheet_name)
    .with_context(|| format!("Failed to load sheet '{sheet_name}'"))?;
```

## Performance

Use `--release` builds for testing. Cargo.toml configured for max optimization (opt-level=3, lto=true). For large files, use `-n` limit.

## Development Workflow

**Branching:**
- Create feature branches for new features: `feature/add-csv-support`
- Create bugfix branches for fixes: `fix/windows-help-popup`
- Submit pull requests to `main` branch
- Direct commits to `main` allowed only for: releases, hotfixes, minor docs

**Commit Guidelines:**

Use conventional commits: `type: description` where type is `feat`, `fix`, `docs`, `refactor`, `test`, or `chore`.

**PR Checklist:**
- [ ] Code compiles, no clippy warnings, formatted
- [ ] Tested with fixtures and real Excel files
- [ ] Updated README.md (if user-facing) or AGENTS.md (if architecture changes)

## Common Patterns (For AI Agents)

**Adding CLI option:** Add field to `Cli` struct in `main.rs`, add clap macros, handle in `main()`

**Adding export format:** Create `export_<format>()` in `display.rs`, add match case in `main.rs`

**Fixing display:** Check `display_table()` in `display.rs`, test with DataTypes sheet

**New cell type:** Add to `CellValue` enum in `workbook.rs`, implement `Display`, update `datatype_to_cellvalue()`

## Useful Commands Reference

```bash
# Full development cycle
cargo fmt && cargo clippy && cargo build --release

# Quick iteration
cargo check && cargo run -- tests/fixtures/test_comprehensive.xlsx -i

# View specific sheet
cargo run -- tests/fixtures/test_tables.xlsx --sheet EmployeesTable

# Test exports
cargo run -- tests/fixtures/test_comprehensive.xlsx --export csv > test.csv
cargo run -- tests/fixtures/test_comprehensive.xlsx --export json > test.json

# Performance check (release mode is crucial)
time ./target/release/xleak tests/fixtures/test_large.xlsx

# Install globally after changes
cargo install --path .
```

## Release Process

xleak uses cargo-dist for automated releases. All distribution channels are now automated. See [RELEASE_CHECKLIST.md](./RELEASE_CHECKLIST.md) for complete instructions. Create a GitHub issue using the "Release" template to track progress.

**Distribution channels:**
- GitHub Releases, Homebrew, Scoop, WinGet, crates.io, AUR (all automated)

**Workflows:**
- `.github/workflows/release.yml` - Main release, Homebrew, crates.io
- `.github/workflows/publish-scoop.yml` - Scoop bucket
- `.github/workflows/publish-aur.yml` - AUR repository
- `.github/workflows/publish-winget.yml` - WinGet manifests

## Questions or Issues?

If implementing features:
1. Check .planning/ (local directory, not tracked in git) for any planning documents
2. Review existing code patterns before adding new patterns
3. Keep changes minimal and focused
4. Prefer editing existing files over creating new ones
5. Test with fixtures in tests/fixtures/ covering multiple formats (.xlsx, .xls, .ods)

---
> Source: [bgreenwell/xleak](https://github.com/bgreenwell/xleak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
