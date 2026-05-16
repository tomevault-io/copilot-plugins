## brave-browser-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Brave Browser Control extension for Claude Desktop that enables browser automation through AppleScript on macOS. It provides tools for tab management, navigation, page interaction, and JavaScript execution via a Model Context Protocol (MCP) server.

## Development Commands

```bash
# Install dependencies
npm install

# Run the MCP server
npm start

# Package the extension (rebuild .dxt file)
npx @anthropic-ai/dxt pack

# Validate manifest before packaging
npx @anthropic-ai/dxt validate manifest.json

# Clean and optimize existing .dxt file
npx @anthropic-ai/dxt clean brave-browser-control.dxt
```

## Architecture

The project implements an MCP server that bridges Claude Desktop with Brave Browser using AppleScript automation.

### Key Components

- **server/index.js**: Main MCP server implementation
  - `BraveControlServer` class handles tool registration and request routing
  - `executeAppleScript()` method executes AppleScript commands via `osascript`
  - Comprehensive error handling for permissions and browser state
  - All 9 browser control tools implemented in the `CallToolRequestSchema` handler

### Tool Implementation Patterns

All tools follow consistent patterns in server/index.js:

1. **Tab ID Resolution**: Most tools accept optional `tab_id` parameter

   - If provided: Iterate through all windows/tabs to find matching ID
   - If omitted: Operate on active tab of front window

2. **AppleScript Execution**:

   - Uses `osascript -e` for AppleScript commands
   - Proper string escaping for JavaScript code injection
   - Error handling for common issues:
     - Permission denied (-1743): Automation permissions required
     - Browser not running (-600): Brave must be launched

3. **JavaScript Injection**:
   - `execute_javascript` tool properly escapes user code
   - `get_page_content` uses custom DOM traversal to preserve link URLs
   - Format: "link text [URL]" for extracted links

### Security Architecture

- Requires macOS automation permissions for Brave Browser
- Can execute arbitrary JavaScript in browser context
- AppleScript strings must be properly escaped to prevent injection
- Extension validates JavaScript URLs to filter out `javascript:void(0)`

### Extension Packaging

- **manifest.json**: DXT extension manifest with tool definitions
- **brave-browser-control.dxt**: Built extension package (zip archive)
- Platform requirement: macOS only (uses AppleScript)
- Runtime requirement: Node.js >= 16.0.0

## GitHub Actions

The project includes Claude Code GitHub Actions:

- **.github/workflows/claude-code-review.yml**: Automated PR review on open/sync
- **.github/workflows/claude.yml**: Interactive Claude assistant triggered by @claude mentions

## Development Notes

- The `.dxt` file should not be committed to version control (added to .gitignore)
- AppleScript error codes are well-documented in the error handling
- Link preservation in `get_page_content` uses optimized array joining vs string concatenation
- All tools return consistent JSON response format via MCP protocol

## Version Management

When updating the extension:
1. Update version in both `manifest.json` and `package.json`
2. Follow semantic versioning: `major.minor.patch`
3. Current version: 0.1.2

---
> Source: [TariqAlagha/brave-browser-control](https://github.com/TariqAlagha/brave-browser-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
