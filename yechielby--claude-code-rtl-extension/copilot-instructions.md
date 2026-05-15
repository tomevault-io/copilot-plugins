## claude-code-rtl-extension

> VS Code extension that adds RTL (Right-to-Left) text support for Hebrew, Arabic and Persian to Claude Code's chat interface. Works in VS Code, Cursor, and Antigravity IDEs.

# Claude Code RTL Extension

## Project Overview

VS Code extension that adds RTL (Right-to-Left) text support for Hebrew, Arabic and Persian to Claude Code's chat interface. Works in VS Code, Cursor, and Antigravity IDEs.

The extension works by injecting CSS and JavaScript into Claude Code's webview files (`webview/index.css` and `webview/index.js`). It creates backups before modifying files and can restore them.

## Tech Stack

- **Language:** TypeScript (strict mode)
- **Build:** esbuild (bundled to single `dist/extension.js`, CJS format)
- **Target:** ES2022, Node16 modules
- **VS Code API:** ^1.94.0
- **No test framework** — no tests exist in this project

## Commands

- `npm run build` — production build
- `npm run watch` — dev build with watch mode
- `npm run package` — create VSIX package via `@vscode/vsce`

## Architecture

Six source files in `src/`:

| File | Purpose |
|---|---|
| `extension.ts` | Entry point. Registers commands, handles auto-reactivation on startup, manages global state (mode + version) |
| `types.ts` | Shared types: `ClaudeExtensionInfo`, `RtlMode`, `RtlStatus` |
| `finder.ts` | Discovers installed Claude Code extensions on disk across platforms (Win/Mac/Linux/WSL) |
| `injector.ts` | Core logic: injects/removes CSS+JS into Claude Code files, manages backups, reports status |
| `content.ts` | All injected CSS rules and JavaScript code as string constants/generators |
| `statusBar.ts` | VS Code status bar item showing current RTL mode |

## RTL Modes

- **Active** (`active`) — CSS scoped to `.YBYrtl` class + toggle button (JS)
- **Always** (`always`) — CSS without class scoping, no JS button
- **Auto** (`auto`) — Per-bubble RTL detection via MutationObserver (JS), `.YBYrtl` applied per element
- **Inactive** (`inactive`) — RTL removed, original files restored from backup

## Key Patterns

- Injected content is wrapped in start/end markers (e.g. `RTL_START_MARKER`/`RTL_END_MARKER`) for clean removal
- Files are backed up as `.bak` before injection; restore from backup on deactivation
- Auto-reactivation runs silently on every activation to re-inject after Claude Code updates
- Version tracking in `globalState` prevents stale CSS from stacking
- The `.YBYrtl` class is the central CSS scoping mechanism — code blocks, tool output, and thinking blocks are forced LTR

## Conventions

- No external runtime dependencies — only VS Code API and Node.js built-ins
- Injected JS uses `var` and ES5 syntax (runs inside webview, not in Node)
- CSS selectors use `[class*="..."]` to match obfuscated class names in Claude Code's webview
- Hebrew/Arabic/Persian detection uses Unicode ranges: `\u0590-\u05FF`, `\u0600-\u06FF`, `\u0750-\u077F`, `\uFB50-\uFDFF`, `\uFE70-\uFEFF`
- When modifying CSS rules in `content.ts`, maintain the pattern of separate `rtlContentRules()` and `ltrOverrideRules()` functions with the prefix parameter

## Publishing

- Publisher: `yechielby` on VS Code Marketplace
- Icon: `claude-code-rtl-logo.png` in root
- VSIX built with `npm run package`

---
> Source: [yechielby/claude-code-rtl-extension](https://github.com/yechielby/claude-code-rtl-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
