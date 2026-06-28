## inpdf

> This file provides context for AI assistants working with this codebase.

# CLAUDE.md

This file provides context for AI assistants working with this codebase.

## Project Overview

**inpdf** is a Rust CLI tool and MCP server for PDF navigation and manipulation. It provides both command-line access and MCP tools for AI assistants to search, read, and extract content from PDFs.

## Architecture

```
src/
├── main.rs           # Entry point, dispatches to CLI commands or MCP server
├── cli.rs            # Clap argument definitions
├── mcp.rs            # MCP server implementation using rmcp
├── page_range.rs     # Page range parser ("1-5,10,15-end" syntax)
├── commands/         # CLI command implementations
│   ├── mod.rs
│   ├── extract.rs    # Extract pages to new PDF
│   ├── grep.rs       # Regex search in PDF text
│   ├── info.rs       # Display PDF metadata
│   ├── merge.rs      # Combine multiple PDFs
│   ├── split.rs      # Split PDF into individual pages
│   └── toc.rs        # Print table of contents
└── pdf/              # Core PDF operations
    ├── mod.rs
    ├── document.rs   # PDF loading/saving wrapper around lopdf
    ├── text.rs       # Text extraction using pdf-extract
    ├── toc.rs        # TOC/bookmark parsing from PDF structure
    └── page_labels.rs # Logical page number mapping (i, ii, 1, 2, etc.)
```

## Key Dependencies

- **lopdf** (0.39): Low-level PDF manipulation (reading, writing, page operations)
- **pdf-extract**: Text extraction from PDFs
- **rmcp** (0.13): MCP server framework
- **clap**: CLI argument parsing
- **tokio**: Async runtime for MCP server

## Common Tasks

### Adding a new CLI command

1. Add the command variant to `Commands` enum in `src/cli.rs`
2. Create `src/commands/yourcommand.rs` with a `pub fn run(...)` function
3. Add `pub mod yourcommand;` to `src/commands/mod.rs`
4. Handle the command in `main.rs` match statement

### Adding a new MCP tool

1. Add a request struct in `src/mcp.rs` if the tool needs multiple parameters
2. Add a method to the `#[tool_router] impl PdfServer` block with `#[tool(description = "...")]`
3. Methods use `Parameters<RequestStruct>` pattern for complex inputs
4. Return a `String` (JSON-serialized result or error message)

### Working with PDFs

- `PdfDocument::open(path)` - Load a PDF
- `doc.page_count()` - Get total pages
- `doc.get_info()` - Get metadata
- `doc.extract_pages(&[1, 2, 3])` - Create new doc with specific pages
- `pdf::text::grep_pdf(path, &regex, max)` - Search text
- `pdf::text::extract_text_pages(path, &pages)` - Get text from pages
- `pdf::toc::extract_toc(path)` - Get bookmarks/TOC

## Page Range Syntax

The `page_range.rs` module parses strings like:
- `"5"` → page 5
- `"1-10"` → pages 1 through 10
- `"10-1"` → pages 10 through 1 (reverse)
- `"5-end"` → page 5 to last page
- `"1-5R"` → pages 1-5 with 90° right rotation
- `"1-3,7,10-end"` → multiple ranges

Use `expand_page_ranges(spec, total_pages)` to get a `Vec<u32>` of page numbers.

## Build & Test

```bash
cargo build              # Debug build
cargo build --release    # Release build
cargo test               # Run tests
cargo run -- <args>      # Run CLI
```

## Known Limitations

- The merge command (`commands/merge.rs`) uses simplified page copying that may not work for all PDFs
- Text extraction quality varies by PDF (scanned documents won't extract text)
- The `tool_router` field in `PdfServer` appears unused but is required by the rmcp macro

---
> Source: [jonhoo/inpdf](https://github.com/jonhoo/inpdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
