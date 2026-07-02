## obsidfluence

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ObsidFluence is an Obsidian plugin that syncs Confluence Cloud pages to Obsidian with proper folder structure preservation. The plugin maintains a hierarchical folder structure that mirrors Confluence's page hierarchy, converting Confluence's HTML/Storage format to Markdown while preserving attachments.

## Build and Development Commands

```bash
# Install dependencies
npm install

# Development mode (watch mode with inline sourcemaps)
npm run dev

# Production build (no sourcemaps, exits after build)
npm run build

# Type checking only (no build)
tsc -noEmit -skipLibCheck
```

The build process uses esbuild (configured in `esbuild.config.mjs`) to bundle `main.ts` into `main.js`.

## Architecture

### Core Components

**ConfluenceSyncPlugin** (main.ts:66)
- Main plugin class extending Obsidian's `Plugin`
- Manages sync lifecycle, settings, and UI interactions
- Coordinates between the Confluence API and Obsidian's file system

**ConfluenceAPI** (main.ts:637)
- Handles all Confluence REST API interactions
- Implements authentication using Basic Auth (email + API token)
- Manages pagination for large result sets
- Key methods:
  - `getSpace()`: Fetch space metadata
  - `getSpacePages()`: Get all pages in a space with pagination
  - `getPageContent()`: Fetch full page content including body.storage
  - `getPageChildren()`: Recursively find child pages (handles multiple API approaches)
  - `getPageAttachments()` / `downloadAttachment()`: Attachment management

**ConfluenceSyncSettingTab** (main.ts:839)
- Settings UI for plugin configuration
- Manages Confluence credentials, space keys, sync intervals, etc.

### Sync Logic Flow

1. **Space Sync** (`syncSpace`, main.ts:169)
   - Creates space folder using sanitized space name
   - Fetches all pages in space
   - Identifies root pages (pages with no ancestors)
   - Recursively syncs each root page and its children

2. **Page Sync** (`syncPage`, main.ts:190)
   - Determines page structure: pages with children become folders with `index.md`, leaf pages become `.md` files
   - Version-based sync optimization: only fetches full content if Confluence version is newer
   - Converts Confluence Storage Format to Markdown
   - Downloads and saves attachments to `attachments/` subfolder
   - Adds frontmatter with metadata (confluenceId, version, lastSynced, etc.)

3. **Content Conversion** (`convertToMarkdown`, main.ts:343)
   - Converts Confluence macros: code blocks, expand panels, info/warning/note panels
   - Handles Confluence images and converts to Markdown image syntax with relative paths
   - Converts Confluence internal links to Obsidian wiki links
   - Converts HTML tables to Markdown tables
   - Converts standard HTML elements (headers, bold, italic, lists, etc.)
   - Decodes HTML entities

### Hierarchical Structure Handling

The plugin preserves Confluence's page hierarchy by:
- **Parent pages with children** → Created as folders with `index.md` containing the page content
- **Leaf pages** → Created as regular `.md` files
- **Attachments** → Stored in `attachments/` subfolder relative to the page

Example structure:
```
Confluence/
└── Space Name/
    ├── Parent Page/
    │   ├── index.md
    │   ├── attachments/
    │   │   └── image.png
    │   └── Child Page.md
    └── Another Page.md
```

### Version Control and Conflict Resolution

- Each synced page includes frontmatter with `confluenceId` and `version`
- Before downloading content, the plugin checks if local version >= Confluence version
- Conflict resolution strategies (configurable):
  - `ask`: Prompt user (currently defaults to not updating)
  - `confluence`: Always use Confluence version
  - `obsidian`: Keep Obsidian version

### Special Conversion Handling

**Expand Macros**: Uses placeholder tokens (`__EXPAND_START__`, etc.) to prevent double-processing during HTML conversion, then converts to Obsidian collapsible callouts at the end.

**Code Blocks**: Extracts language and CDATA content from Confluence structured macros.

**Internal Links**: Converts `<ac:link>` with `<ri:page>` to Obsidian wiki links: `[[Page Title]]` or `[[Page Title|Link Text]]`

**Images**: Converts attachment references to relative paths: `![filename](attachments/sanitized-filename.png)`

## Key Implementation Details

- **Pagination**: API calls use `start` and `limit` parameters to handle large result sets (limit=100)
- **Child Page Discovery**: The `getPageChildren` method tries multiple API endpoints (child/page, child/folder, descendant/page) to handle different Confluence configurations
- **File Sanitization**: Characters `\ / : * ? " < > |` are replaced with `-` in filenames
- **HTML Entity Decoding**: Comprehensive entity map including common Unicode characters and numeric entities
- **Attachments**: Downloaded as binary using `vault.createBinary()` or `vault.modifyBinary()`

## Common Pitfalls

- **Children Not Found**: Confluence's child/page API sometimes returns empty results. The plugin falls back to checking descendants and filtering for direct children based on ancestors.
- **Expand Macro Processing**: Must use placeholder approach to avoid nested tag issues during regex replacements.
- **Path Construction**: Ensure attachment paths are relative to the content file location (affects both folders and regular files differently).
- **Version Comparison**: Always check existing version before fetching full content to minimize API calls.

---
> Source: [NinDaniel/ObsidFluence](https://github.com/NinDaniel/ObsidFluence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
