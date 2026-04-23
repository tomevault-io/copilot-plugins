## raycast-motion-mover

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Development and build commands:
- `npm run dev` - Start Raycast extension development mode
- `npm run build` - Build the extension for production
- `npm run lint` - Run ESLint to check code quality
- `npm run fix-lint` - Automatically fix linting issues
- `npm run publish` - Publish extension to Raycast Store

## Architecture

This is a Raycast extension that provides Vim-style window management on macOS. The extension consists of:

**Core Structure:**
- Single command extension with one main function in `src/move.ts`
- Command accepts text input following Vim-style syntax patterns
- Uses Raycast's WindowManagement API to manipulate window positions

**Command Syntax:**
The extension parses commands with regex pattern: `^w(?<screen>\d+|_)?(?<type>[fchqt])(?<pos>[A-Z]{0,2})?$`
- `screen`: Screen number (1, 2, etc.) or `_` for active screen
- `type`: Zone type - 'f' (fullscreen), 'c' (center), 'h' (half), 't' (third), 'q' (quarter)
- `pos`: Position - 'L'/'R' (left/right), 'TL'/'TR'/'BL'/'BR' (corners), 'C' (center)

**Implementation:**
Complete window positioning logic implemented in `calculateWindowBounds()` function that handles all zone types and positions based on screen dimensions.

**Error Handling:**
Uses Raycast's toast notifications for errors and HUD for success messages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lalunastudio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
