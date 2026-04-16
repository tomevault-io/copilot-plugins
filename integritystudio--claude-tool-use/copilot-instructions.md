## claude-tool-use

> Static HTML file explorer for browsing Claude Code session history, debug logs, and project data.

# Claude History Explorer

Static HTML file explorer for browsing Claude Code session history, debug logs, and project data.

## Project Structure

```
claude-tool-use/
├── index.html              # Main explorer interface
├── css/styles.css          # Styling with CSS custom properties
├── js/
│   ├── data.js             # Generated file structure data
│   ├── utils.js            # Utility functions, icons, helpers
│   ├── renderers.js        # Preview renderers (JSONL, JSON, code, etc.)
│   └── app.js              # Main application logic
├── scripts/
│   └── generate-structure.js  # Node.js script to regenerate data.js
├── CLAUDE.md               # This file
└── .claude/
    └── settings.local.json
```

## Quick Start

```bash
# Open in browser
open index.html

# Regenerate data after adding new history files
node scripts/generate-structure.js > js/data.js
```

## Features

### File Browsing
- **Grid/Tree Views**: Toggle between card grid and hierarchical tree
- **Search**: Real-time filtering of files and folders
- **Breadcrumb Navigation**: Home icon, chevron separators, path truncation, tooltips
- **Stats Bar**: Folder/file counts, total size (includes truncated files)

### JSONL Rendering
- Parses Claude session logs (`.jsonl` files)
- Displays entries with type badges (summary, user, assistant, etc.)
- Extracts meaningful summaries from message content
- **Clickable entries**: Opens full JSON in new styled page

### File Type Support
- **JSONL**: Session logs with entry parsing
- **JSON**: Syntax-highlighted formatting
- **Code**: Syntax highlighting for JS/TS/etc.
- **Markdown**: Rendered preview with source
- **Text**: Plain text display

### UX Polish
- Display names hide `-Users-username-` prefixes
- Truncation indicators for folders with hidden files (`+N more`)
- Hover states and visual feedback
- ARIA accessibility labels

## Data Generation

The `scripts/generate-structure.js` script scans the directory and generates `js/data.js`:

```bash
node scripts/generate-structure.js > js/data.js
```

### Configuration (in generate-structure.js)
- `SKIP_DIRS`: Directories to exclude (node_modules, .git, css, js, etc.)
- `SKIP_FILES`: Files to exclude (index.html, package.json, etc.)
- `LARGE_DIRS`: Directories with reduced file limits (debug, todos, etc.)
- `MAX_FILES_PER_DIR`: Default 20 files shown per folder
- `MAX_FILES_LARGE_DIR`: 10 files for large directories

## Tech Stack

- Pure HTML/CSS/JavaScript (no build tools)
- CSS Grid for responsive layout
- CSS custom properties for theming
- Blob URLs for dynamic JSONL entry pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/integritystudio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
