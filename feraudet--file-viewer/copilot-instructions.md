## file-viewer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

File Viewer is a local HTTP server (Go) that renders Markdown, JSON, HTML, and text files with a modern web interface. It's designed for iTerm2's browser pane integration via `~/bin/open-in-iterm-browser` (Cmd+click on files).

## Build & Run

```bash
go build -o file-viewer
./file-viewer
```

Server runs on `http://localhost:4120`

## Endpoints

- `GET /{filepath}` - Render file (Markdown, JSON, HTML, text)
  - **Special**: Static resources (images, CSS, JS, fonts) served RAW when requested from HTML (Referer detection)
- `GET /?path=X&filename=Y` - Alternative file access with path context
- `GET /` - Shows Claude Code CHANGELOG (fetched from GitHub)
- `GET /asset?path=/absolute/path` - Serve static assets (images, SVG, PDF)
- `GET /mtime/{filepath}` - Return file modification time (for live reload)

## Architecture

Single-file Go application (`main.go`) with embedded HTML/CSS/JS template:

**Key functions:**
- `handler()` - Main HTTP router (line 225)
- `renderFile()` - Dispatch to format-specific renderers (line 336)
- `renderMarkdown()` - Custom Markdown parser with TOC generation (line 377)
- `renderJSON()` - Interactive tree view with expand/collapse (line 725)
- `buildHTML()` - Generate full HTML page with all dependencies (line 852)

**Markdown renderer features:**
- Inline processing: bold, italic, code, links, images, math, emoji
- Block elements: headers with anchors, lists (ordered/unordered/task), tables, blockquotes, code blocks
- Special blocks: Mermaid diagrams, KaTeX math (`$$...$$` and `$...$`)
- Auto-generated TOC for documents with 3+ headers

**Frontend dependencies (CDN):**
- Prism.js for syntax highlighting
- KaTeX for math rendering
- Mermaid for diagrams

## Supported Formats

| Extension | Handler |
|-----------|---------|
| .md, .markdown | Markdown with TOC, syntax highlighting, math, diagrams |
| .json | Interactive tree with search, expand/collapse |
| .yaml, .yml | YAML with syntax highlighting |
| .toml | TOML with syntax highlighting |
| .tf | Terraform/HCL with syntax highlighting and copy button |
| .csv | Interactive table with filter |
| .html, .htm | Raw HTML passthrough (no wrapper) | Served directly with Content-Type: text/html |
| .txt, .text | Preformatted text with search |

## HTML Static Resource Serving

Static resources referenced by HTML files are automatically served in RAW mode via HTTP Referer detection:

**Helper functions:**
- `getStaticContentType(ext string) (string, bool)` (line ~300) - Returns Content-Type for static extensions
- `isRefererFromHTML(referer string, host string) bool` (line ~345) - Validates referer from HTML on same host

**Workflow:**
1. User opens `http://localhost:4120/page.html`
2. Browser requests `image.png` with `Referer: .../page.html`
3. `isRefererFromHTML()` validates referer (same host, .html/.htm)
4. `getStaticContentType()` checks if extension is static
5. File served raw with proper Content-Type

**Security:**
- Only same-host referers accepted
- External referers fall back to wrapped rendering

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/feraudet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/feraudet)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
