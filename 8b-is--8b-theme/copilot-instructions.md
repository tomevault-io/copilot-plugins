## 8b-theme

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **8b-theme**, a VSCode color theme extension providing a "Retro Cyberpunk Dark Theme" aesthetic. It's a straightforward theme extension with no build process or compilation steps.

## Project Structure

- `package.json` - Extension manifest defining the theme contribution point
- `themes/8b-color-theme.json` - Complete theme definition with colors and tokenColors
- No source code to compile - pure JSON configuration

## Development Workflow

### Testing the Theme

Press `F5` in VSCode to launch the Extension Development Host with the theme loaded. Changes to the theme JSON file are automatically applied to the development window.

To select the theme in the dev window:
- Use `File > Preferences > Theme > Color Theme` menu
- Or command palette: `Preferences: Color Theme` (Ctrl/Cmd+K Ctrl/Cmd+T)

### Inspecting Token Scopes

When modifying `tokenColors` in the theme file, use the `Developer: Inspect Editor Tokens and Scopes` command (Ctrl/Cmd+Shift+P) to see which scopes are being applied to specific code elements.

## Theme Architecture

The theme follows VSCode's TextMate-based color theme structure:

- **`colors`** object: Defines UI colors (editor background, sidebar, activity bar, etc.)
- **`tokenColors`** array: Defines syntax highlighting rules mapped to TextMate scopes

Each `tokenColors` entry contains:
- `name`: Descriptive label
- `scope`: Array of TextMate scope selectors
- `settings`: `foreground`, `background`, `fontStyle` properties

## Installation

To install locally for personal use:
```bash
cp -r . ~/.vscode/extensions/8b-theme-0.0.1/
```

Then restart VSCode.

## Publishing

When ready to share publicly, follow the [VSCode extension publishing guide](https://code.visualstudio.com/api/working-with-extensions/publishing-extension).

---
> Source: [8b-is/8b-theme](https://github.com/8b-is/8b-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
