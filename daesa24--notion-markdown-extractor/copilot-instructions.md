## notion-markdown-extractor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Python CLI tool that extracts Notion pages to Markdown with automatic project organization. Built for Claude Code workflows - extracts content to `user-context/notion-pages/` for easy context loading.

**Status:** MVP Complete (v0.1.0)

## Build & Development Commands

All commands run from `execute/` directory:

```bash
# Install dependencies
uv sync

# Install CLI globally (for testing)
uv tool install .

# Run tests
uv run pytest
uv run pytest --cov=src              # with coverage
uv run pytest tests/test_cli.py      # single file

# Run CLI in development mode
uv run python -m notion_markdown_extractor extract <url>
uv run python -m notion_markdown_extractor status
uv run python -m notion_markdown_extractor configure --token <token>

# Uninstall global CLI
uv tool uninstall notion-markdown-extractor
```

## Architecture

Pipeline architecture with 6 core modules in `execute/src/notion_markdown_extractor/`:

```
CLI (cli.py) - Click commands, orchestrates extraction flow
    ↓
NotionClient (notion_client.py) - API wrapper, fetches pages/blocks
    ↓
BlockConverter (block_converter.py) - Transforms Notion blocks → Markdown
    ↓
Storage (storage.py) - File I/O, image downloads

Config (config.py) - Token persistence (~/.notion-md/config.yaml)
ProjectDetector (project_detector.py) - Auto-directory detection
```

**Data flow:** URL → extract page ID → fetch metadata → fetch blocks recursively → convert to markdown → save file + download images

## Key Implementation Details

**Token priority:** Environment variable `NOTION_API_TOKEN` → config file `~/.notion-md/config.yaml`

**Output paths:**
- Default: `./user-context/notion-pages/<kebab-case-title>.md`
- Images always: `./images/` relative to markdown file

**API resilience:** Exponential backoff for rate limiting (429 errors) - 1s → 2s → 4s, max 3 retries

**Synced blocks:** Recursively fetches content from source blocks, processes children transparently

**Filename sanitization:** Converts to kebab-case, strips special characters

## Supported Block Types

**Full support:** paragraphs, headings (1-3), quotes, bulleted/numbered lists, code blocks, images, tables, callouts, toggles, dividers, file attachments

**Partial:** synced blocks (content extracted), internal links (placeholder text)

**Not supported:** databases, embeds, video/audio, column layouts

## Project Structure

```
execute/                    # Implementation directory (run commands here)
├── src/notion_markdown_extractor/  # Python package
├── tests/                  # Test suites (pytest)
└── pyproject.toml         # Package config (UV build system)
docs/                       # Documentation
├── architecture.md        # System design details
├── notion-api-docs/       # Crawled Notion API reference
└── user-docs/             # Usage guides
```

---
> Source: [DAESA24/notion-markdown-extractor](https://github.com/DAESA24/notion-markdown-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
