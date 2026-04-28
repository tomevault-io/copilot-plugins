## read-in-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RICLI is a terminal-based ebook reader supporting EPUB and TXT formats. It uses the `blessed` library for terminal UI rendering and organizes its code into four layers: CLI/entry, parser, store, and UI.

## Commands

```bash
npm start              # Dev mode - data stored in ~/.ricli-dev
npm run release:patch  # Release 1.x.x
npm run release:minor  # Release x.0.0
npm run release:major  # Release x.0.0
npm run release:dry    # Dry run without publishing
```

Releases use Conventional Commits and auto-generate CHANGELOG.md via release-it.

## Architecture

### Entry & CLI (`src/index.js`)
- Uses `commander` for CLI argument parsing
- Two entry points: direct file path (`ricli book.epub`) or interactive bookshelf (`ricli`)
- `config` subcommand for viewing/modifying/resetting configuration
- Routes between library UI and reader based on arguments

### Parser Layer (`src/parser/`)
- `index.js` - dispatcher that routes to correct parser based on file extension
- `epub-parser.js` - uses `epub2` library, converts HTML content to plain text via `html-to-text`
- `txt-parser.js` - parses plain text with automatic chapter detection via regex (looks for chapter markers like "Chapter", "第X章", etc.)
- All parsers return `{ title, chapters: [{ title, content }], filePath }`

### Store Layer (`src/store/`)
- `paths.js` - defines data directory (`~/.ricli/` or `~/.ricli-dev/` in dev mode) and file paths. **Dev/prod isolation via `NODE_ENV`** is critical: set to `development` to use `~/.ricli-dev/`.
- `config.js` - JSON config with theme colors (fg/bg for body, header, toc-selected), padding, linesPerPage
- `library.js` - bookshelf management (records opened books in `library.json`)
- `progress.js` - reading position persistence (chapter index + scroll position per book)

### UI Layer (`src/ui/`)
- `screen.js` - creates blessed `Screen` instance with terminal dimensions
- `book-select.js` - interactive bookshelf with book list and delete/open actions
- `reader-view.js` - main reading view with header (title/chapter), scrollable content, footer (keybindings/progress). Chapter navigation and scroll position are persisted.
- `toc-view.js` - overlay table of contents modal
- `input-prompt.js` - modal for entering file paths

### Key Bindings (reader-view.js:128-182)
- Scroll: `↑↓/jk`, Page: `PgUp/PgDn/Space/b`, Chapter: `←→/hl`, TOC: `t`, Quit: `q/Esc`
- `g`/`G` for chapter start/end

## Technical Notes

- **Terminal UI**: All UI is rendered via `blessed` library (not stdout printing). Key handlers are attached to the screen object.
- **Progress persistence**: Called on every scroll/navigation action via `persistProgress()`. Must call `screen.unkey(...)` to remove listeners before exiting to prevent key handler leaks (reader-view.js:171-180).
- **No test suite**: `npm test` exits with error. Add tests if needed.
- **Node version**: Requires `>=18.0.0`

test scripts: npm start -- test-book.txt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alfxjx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
