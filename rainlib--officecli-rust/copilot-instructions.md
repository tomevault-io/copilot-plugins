## officecli-rust

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

OfficeCLI is a single-binary, pure-Rust CLI for creating, reading, modifying, and rendering Office documents (.docx, .xlsx, .pptx) and PDFs. No Office installation needed. It also includes an MCP server for AI agent integration and a watch server for live HTML previews.

## Build & Development Commands

```bash
cargo build                     # Debug build
cargo build --release           # Release build
cargo check                     # Fast compile check (no binary)
cargo test                      # Run all tests (inline unit tests only, no integration test dirs)
cargo clippy --all-targets -- -D warnings  # Lint
cargo fmt -- --check            # Check formatting
cargo fmt                       # Auto-fix formatting
cargo run -- <ARGS>             # Run with CLI args
make smoke                      # Smoke test: create + view a doc, verify binary works
make lint                       # fmt + clippy combined
```

Tests are inline (`#[test]` within source files), not in separate `tests/` directories. No integration tests exist.

## Workspace Structure

Cargo workspace with 7 crates under `crates/`:

| Crate | Purpose |
|-------|---------|
| `officecli` | CLI binary (entry point) |
| `handler-common` | `DocumentHandler` trait + shared types (`DocumentNode`, `HandlerError`, `PathSegment`, `TextOffsetMap`, etc.) |
| `oxml` | OOXML ZIP package abstraction (`OxmlPackage` — open, read/write parts, content types, relationships) |
| `docx-handler` | `WordHandler` — Word DOM (`WordDom`/`WordNode`) over `OxmlPackage` |
| `xlsx-handler` | `ExcelHandler` — workbook model (`WorkbookModel`/`Cell`) over `OxmlPackage` |
| `pptx-handler` | `PptxHandler` — presentation/slide XML over `OxmlPackage` |
| `pdf-handler` | `PdfHandler` — separate backend (lopdf, custom content stream parser, font embedder, modifier) |

Dependency graph: `handler-common` is the root trait crate. `oxml` depends on `handler-common`. OOXML handlers depend on `oxml`. `pdf-handler` depends only on `handler-common`. `officecli` depends on all handlers.

## Three-Layer Architecture

All handlers implement the same `DocumentHandler` trait with three layers:

- **L1 (Semantic/Read)**: `view` modes (text, annotated, outline, stats, issues, html, svg)
- **L2 (DOM/Query)**: `get`, `query`, `set`, `add`, `remove`, `move` — path-based element operations
- **L3 (Raw)**: `raw`, `raw-set`, `validate` — direct XML/XPath access

Always prefer higher layers. L2 handles structured edits; L3 is for cases L2 can't express.

## Key Architectural Patterns

- **Interior mutability**: All handlers use `RefCell` (`RefCell<OxmlPackage>` or `RefCell<PdfReader>`) because `DocumentHandler` trait methods take `&self`.
- **Parse-modify-serialize**: OOXML handlers parse XML into a custom DOM tree, mutate it, serialize back to XML string, then write back to the ZIP package.
- **Path-based addressing**: XPath-like paths with 1-based indexing (e.g., `/body/p[3]`, `/slide[1]/shape[@id=550950021]`). Alias resolution built in (e.g., "paragraph" → "p").
- **TextOffsetMap**: Every format emits full text + character-offset-to-path-ID mapping for AI agents to precisely locate and modify text.
- **Inline templates**: Blank document creation uses hardcoded XML strings, no external template files.
- **Platform conditional**: Resident mode (Unix Domain Socket IPC) is Unix-only; stubbed with error on other platforms.

## Handler Module Structure

Each handler crate follows a consistent module pattern:

- `handler.rs` — `DocumentHandler` trait impl + XML parse/serialize
- `dom_types.rs` — Format-specific DOM types and element enums
- `navigation.rs` — Path parsing and DOM tree navigation
- `mutations.rs` — `set`, `move`, `remove` operations
- `add.rs` — `add` element operations
- `view.rs` — View mode implementations
- `query.rs` — CSS-like selector query
- `raw.rs` — Raw XML read/write
- `html_preview.rs` — HTML rendering for watch/preview
- `text_offset.rs` — `extract_text_with_offsets`

Format-specific extras: docx has `para_id.rs`, `helpers.rs`; xlsx has `formula_eval.rs`; pptx has `svg_preview.rs`; pdf has `content_stream.rs` (~108KB parser/modifier), `font_embedder.rs`, `modifier.rs`, `reader.rs`, `render.rs`, `text_extract.rs`.

## CLI Entry Point

`crates/officecli/src/main.rs` — clap-based CLI. `open_handler()` routes by file extension (.docx → WordHandler, .xlsx → ExcelHandler, .pptx → PptxHandler, .pdf → PdfHandler). Commands are defined in `crates/officecli/src/commands/mod.rs`.

Additional modes:
- **Resident mode**: Background IPC server via Unix Domain Socket (auto-starts on first access, 60s idle)
- **Watch mode**: Axum HTTP server for live HTML preview (default port 26315)
- **MCP server**: Stdio-based JSON-RPC 2.0 server exposing 13 tools

## Contribution Rules

From CONTRIBUTING.md — two hard rules:

1. **One PR = one atomic change**. Cannot be further decomposed. If a diff can be split into multiple independent PRs, submit each separately.
2. **Every PR must include a verifiable validation method**. Bug fixes: officecli command sequence showing before/after. Features: screenshot + command sequence.

## Important Conventions

- Paths are **1-based** (XPath convention), but `--index` is **0-based** (array convention). Excel row/col `--index` is **1-based** (OOXML convention).
- Always quote paths with brackets in shell: `'/slide[1]'` to avoid zsh/bash glob expansion.
- All attributes go through `--prop key=value`, not separate `--name` flags.
- When unsure about property names or command syntax, use the help system: `officecli help <format> <element>` rather than guessing.

---
> Source: [RainLib/OfficeCLI-rust](https://github.com/RainLib/OfficeCLI-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
