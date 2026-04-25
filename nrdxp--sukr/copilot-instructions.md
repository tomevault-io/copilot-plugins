## sukr

> This project uses [predicate](https://github.com/nrdxp/predicate) for agent configuration.

# Project Agent Configuration

## Predicate System

This project uses [predicate](https://github.com/nrdxp/predicate) for agent configuration.

> [!IMPORTANT]
> You **must** review [.agent/PREDICATE.md](.agent/PREDICATE.md) and follow its instructions before beginning work.

**Active Personas:**

- Rust idioms (`.agent/personas/rust.md`)
- DepMap MCP tools (`.agent/personas/depmap.md`)
- Personalization (`.agent/personas/personalization.md`)

---

## Project Overview

**sukr** is a minimal static site compiler written in Rust. Suckless, zero JS, transforms Markdown into high-performance static HTML.

### Philosophy

- **Suckless:** No bloated runtimes, no unnecessary JavaScript
- **Hermetic:** Single binary with all dependencies compiled in
- **Elegant:** Syntax highlighting via Tree-sitter, templates via Tera

### Architecture

The compiler implements an **Interceptor Pipeline**:

1. **Ingest:** Walk `content/`, parse TOML frontmatter
2. **Stream:** Feed Markdown to `pulldown-cmark` event parser
3. **Intercept:** Route code blocks to Tree-sitter, Mermaid, pulldown-latex
4. **Render:** Push modified events to HTML writer
5. **Layout:** Wrap in Tera templates (runtime, user-customizable)
6. **Write:** Output to `public/`

---

## Build & Commands

```bash
# Development
nix develop          # Enter dev shell with Rust toolchain
cargo build          # Build compiler
cargo run            # Run compiler (builds site to public/)

# Production
nix build            # Build hermetic release binary
./result/bin/sukr    # Run release compiler

# CLI Usage
sukr                           # Build with ./site.toml
sukr -c sites/blog/site.toml   # Build with custom config
sukr --help                    # Show usage
```

---

## Code Style

- Rust 2024 edition
- Follow `.agent/personas/rust.md` conventions
- Prefer standard library over external crates
- No `unwrap()` in library code; use proper error handling

---

## Architecture

```
.
├── Cargo.toml           # Rust manifest
├── flake.nix            # Nix build environment
├── site.toml            # Site configuration (or in sites/*)
├── src/
│   ├── main.rs          # Pipeline orchestrator
│   ├── config.rs        # TOML config loader
│   ├── content.rs       # Content discovery, sections
│   ├── template_engine.rs # Tera template engine
│   ├── feed.rs          # Atom feed generation
│   ├── highlight.rs     # Tree-sitter highlighting
│   └── render.rs        # Pulldown-cmark interception
├── templates/           # Tera templates (base, page, section/*)
├── content/             # Markdown + TOML frontmatter
├── static/              # CSS, images, _headers
└── public/              # Generated output
```

---

## Testing

- Test runner: `cargo test`
- Naming: `test_<scenario>_<expected_outcome>`
- Focus on content transformation correctness

---

## Security

- No user input at runtime (build-time only)
- Validate frontmatter schema during parsing
- No secrets in content or templates

---

## Configuration

Site configuration lives in `site.toml`:

```toml
title    = "My Site"
author   = "Author Name"
base_url = "https://example.com"

[paths]  # All optional, defaults shown
content   = "content"
output    = "public"
static    = "static"
templates = "templates"

[nav]
nested = false  # Hierarchical sidebar navigation
toc    = true   # Table of contents on pages

[feed]
enabled = true  # Generate Atom feed

[sitemap]
enabled = true  # Generate sitemap.xml
```

---
> Source: [nrdxp/sukr](https://github.com/nrdxp/sukr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
