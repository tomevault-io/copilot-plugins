## workspace-colors

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VS Code extension that applies color themes to workspace UI elements (title bar, status bar, tabs) to help distinguish between different VS Code windows.

## Build Commands

```bash
# VS Code Extension (in vscode-workspace-colors/)
npm run compile          # Compile TypeScript
npm run watch            # Watch mode
./build.sh               # Build and package .vsix
./build.sh --publish     # Build and publish to Open VSX (requires OVSX_PAT env var)

# Python TUI (root directory)
python color_picker.py   # Interactive terminal color picker (requires textual)
```

## Architecture

**VS Code Extension** (`vscode-workspace-colors/`):
- `src/extension.ts` - Entry point, registers commands, creates webview color picker UI
- `src/colorUtils.ts` - Color manipulation (hex↔HSL conversion, brightness adjustment)
- `src/themeApplier.ts` - Writes color customizations to VS Code workspace settings
- `src/colors.json` - Color palette organized by category (reds, oranges, yellows, etc.)

**Python TUI** (`color_picker.py`):
- Standalone Textual app that reads the same `colors.json` and applies themes to `.vscode/settings.json`
- Provides live preview as you navigate colors

## Publishing

- **Open VSX**: https://open-vsx.org/extension/OhadRubin/workspace-colors
- **GitHub Actions**: Triggers on `v*` tags or manual dispatch
- **Secrets needed**: `OVSX_PAT` (Open VSX), `VSCE_PAT` (VS Code Marketplace)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OhadRubin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
