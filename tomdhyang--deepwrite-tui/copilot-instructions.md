## deepwrite-tui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Deepwrite is a terminal-based Markdown writing tool written in Rust. It combines Yazi-style file browsing with iA Writer-style focused writing. Target users are "vibe coders" — non-technical users who work with Markdown documentation via AI tools like Claude Code.

## Build & Development Commands

```bash
cargo build                    # Debug build
cargo build --release          # Release build
cargo run -- [path]            # Run (optional path to dir or file)
cargo test                     # Run all tests
cargo test <name>              # Run specific test (e.g. cargo test word_count)
cargo test -- --nocapture      # Run tests with stdout visible
cargo fmt                      # Format code
cargo clippy                   # Lint
```

This is a Cargo workspace with two members: the root `deepwrite` package and `crates/edtui` (a forked text editor widget).

## Architecture

**Two-mode state machine** — `App` (`src/app.rs`) drives everything via an immediate-mode event loop:
- `AppMode::Browse` — file browser on left panel (30 cols), keyboard-driven navigation
- `AppMode::Edit` — non-modal text editor (Emacs-style, not vim), auto-collapses browser in Focus Mode

**Module layout** (`src/lib.rs` exposes all):

| Module | Purpose |
|--------|---------|
| `browser/` | File browser — `Navigator` walks directories, `entries` lists files, `actions` handles CRUD, `widget` renders with prompt overlays |
| `editor/` | `EditorWrapper` wraps the forked `edtui` crate. Submodules: `keymap` (non-modal bindings), `markdown` (regex-based syntax highlighting), `formatting` (bold/italic/heading shortcuts), `focus` (sentence/paragraph/typewriter dimming), `sentence` (boundary detection), `word_count` (CJK-aware) |
| `services/` | `auto_save` (debounced 2s writes via temp+rename), `file_io` (encoding detection with chardetng), `file_watcher` (notify-based external change detection) |
| `ui/` | `layout` (dual-panel geometry), `status_bar` (word count + mode label) |
| `theme.rs` | Light/dark color system with system detection, focus-mode dimming |
| `config.rs` | `~/.config/deepwrite/config.toml` — editor, focus, theme, browser settings |

**Entry point:** `src/main.rs` — CLI parsing (clap), terminal setup (crossterm raw mode + alternate screen), config loading, then `App::run()`.

## Key Design Decisions

- **edtui is forked** at `crates/edtui/` — it's a local workspace member, not a registry dependency. Changes to the editor widget go there.
- **Regex highlighting, not tree-sitter** — tree-sitter-md caused SIGBUS crashes; regex is simpler and works.
- **Custom sentence detection** — `unicode-segmentation` was insufficient for Chinese/English mixed text; custom implementation in `editor/sentence.rs`.
- **Non-modal editing** — Users type immediately. Arrow keys, not `hjkl`. Emacs-style shortcuts (Ctrl+A/E for home/end, etc.).

## Testing

Integration tests live in `tests/` and cover config parsing, browser entries, markdown formatting, sentence detection, and word counting. Tests use `tempfile` for fixtures.

## Config

User config at `~/.config/deepwrite/config.toml` with sections: `[editor]`, `[focus]`, `[theme]`, `[browser]`. Defaults are hardcoded in `config.rs`.

---
> Source: [tomdhyang/deepwrite-tui](https://github.com/tomdhyang/deepwrite-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
