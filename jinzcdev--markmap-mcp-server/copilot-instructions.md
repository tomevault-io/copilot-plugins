## markmap-mcp-server

> This file provides guidance to agents when working with code in this repository.

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Commands

```bash
# Build (TypeScript → build/)
npm run build

# Run tests
npm test                      # vitest run (single pass)
npm run test:watch            # vitest in watch mode

# Development
npm run dev                   # tsc-watch → auto-restart on changes

# Formatting
npm run format                # prettier --write (also runs via lint-staged on commit)

# Run the server locally
node build/index.js
# With options:
node build/index.js --output /tmp/markmap --open always --return-mode both --offline
```

## Architecture

This is an **MCP server** (Model Context Protocol) that converts Markdown to interactive mind maps using the `markmap` family of libraries. It runs over stdio transport.

### Entry point: `src/index.ts`

1. Parses CLI args (`minimist`) and env vars into a `MarkmapMcpContext` (output dir, open mode, return mode, offline flag)
2. Creates an `McpServer` instance, registers tools via `registerMarkmapTools()`, and connects to `StdioServerTransport`

### Tool registration: `src/mcp/tools/`

- **`context.ts`** — Type definitions for `MarkmapMcpContext`, `OpenMode` (`"always" | "never" | "agent"`), `ReturnMode` (`"path" | "content" | "both"`)
- **`registry-base.ts`** — Abstract `RegistryBase` with `register()` method
- **`tool-registry.ts`** — `ToolRegistry extends RegistryBase`, adds `registerTools()` wrapper
- **`markmap-tools.ts`** — `MarkmapToolRegistry` implements 4 tools and 1 prompt:
  - `markdown_to_mindmap` — main conversion tool (html/png/svg/jpg)
  - `list_mindmaps` — list generated files
  - `get_mindmap` — retrieve a single file by path (with path traversal protection)
  - `cleanup_mindmaps` — delete old/all files
  - `mindmap_from_content` — prompt that asks the LLM to structure content before calling the tool

### Core markmap logic: `src/markmap/`

- **`createMarkmap.ts`** — Uses `markmap-lib` (Transformer) and `markmap-render` (fillTemplate) to convert Markdown → HTML. Injects the `markmap-toolbar`, an export toolbar (PNG/JPG/SVG/Copy Markdown via `html-to-image`), and the original markdown source. In `--offline` mode, inlines all JS/CSS assets.
- **`exportImage.ts`** — Launches headless Chromium via Playwright to render HTML → PNG/JPG/SVG. Requires `playwright` optional dependency.
- **`lifecycle.ts`** — File listing, retrieval (with path traversal check), and cleanup for the output directory.
- **`escape.ts`** — HTML/text escaping for safe embedding.
- **`i18n.ts`** — English toolbar labels used in the generated HTML.

### Return mode system

The server's `returnMode` controls what the MCP client receives:

- **`path`** — `{htmlFilePath, filePath}` JSON only
- **`content`** — Inline HTML (text) or base64 image block; falls back to paths if HTML ≥ 200KB
- **`both`** — Paths JSON + inline content

### Open mode system

The server's `open` setting controls whether the browser opens:

- **`always`** / **`never`** — fixed server-side
- **`agent`** — the `open` parameter is exposed on the tool, letting the LLM decide per call

## Key patterns

- **Tool registry pattern**: `RegistryBase` → `ToolRegistry` → `MarkmapToolRegistry`. Each tool is registered in its own `register*()` method.
- **Server config is set at startup** (CLI/env), not per-tool-call. The tool description text adapts to the server's configuration.
- **File naming**: Generated files are prefixed with `markmap-` so `list_mindmaps` and `cleanup_mindmaps` can discover them.
- **Tests** use vitest with `describe`/`it`/`expect` in `tests/unit/`. Tests import from `../../src/` directly (no path aliases).
- **No eslint config file** — the project uses `eslint.config.js` (flat config format) with TypeScript ESLint and Prettier integration.
- **Husky + lint-staged** runs prettier and eslint on staged files pre-commit.

---
> Source: [jinzcdev/markmap-mcp-server](https://github.com/jinzcdev/markmap-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
