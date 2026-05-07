## extension-architecture

> 1. `esbuild` for extension host (`src/extension.ts` → `dist/extension.js`, Node context)

# Extension Architecture

## Build Pipeline
Two separate builds:
1. `esbuild` for extension host (`src/extension.ts` → `dist/extension.js`, Node context)
2. `esbuild` for webview (`webview/index.ts` → `dist/webview.js`, browser context)

These are DIFFERENT runtimes. The extension host has Node + VS Code API. The webview has DOM + Canvas.
They communicate only via `postMessage` / `onDidReceiveMessage`.

## Webview Security
CSP requires `script-src 'nonce-...'`. The nonce is generated per webview instance in the extension host.
The webview HTML template injects the nonce into the script tag.
No inline scripts, no eval, no external CDN scripts.

## Agent Detection — Two Modes

### Mode 1: Transcript Parsing (fallback)
- Cursor stores transcripts at `~/.cursor/projects/<workspace-hash>/agent-transcripts/`.
- `cursorWatcher.ts` scans `~/.cursor/projects/*/agent-transcripts/` with glob.
- Each JSONL line is `{ role, content }`. Last few lines → heuristic activity inference.
- User line after assistant lines → immediate idle. 8-second silence → idle.

### Mode 2: Cursor Hooks (preferred)
Cursor has a hooks API (`~/.cursor/hooks.json`) that fires shell scripts on agent events.

**Hook events we use:**
- `preToolUse` — tool_name maps to activity: Read/Glob → reading, Write/StrReplace → editing, Shell → running, Grep/SemanticSearch → searching, Task → phoning
- `stop` — `status: "completed"|"aborted"|"error"`. Completed + did work → celebrate. Error → error reaction. Aborted → idle.
- `beforeSubmitPrompt` — user sent message → idle
- `subagentStart` → phoning (character picks up desk phone)
- `subagentStop` → back to previous work activity

**Files involved:**
- `hooks/cursor-office-hook.sh` — shell script bundled with extension. Writes JSON to `/tmp/cursor-office-state.json`.
- `src/hooksInstaller.ts` — installs/removes hooks from `~/.cursor/hooks.json`. Merges gently: checks for `cursor-office-hook` marker, only adds our entries, preserves existing hooks.
- `cursorWatcher.ts` — on startup checks if hooks state file exists. If yes, watches it instead of transcripts.

**Commands:**
- `Cursor Office: Enable Hooks` — installs hooks, tells user to restart Cursor.
- `Cursor Office: Disable Hooks` — removes only our hook entries, leaves others intact.

**Hook installation pattern** (learned from Honcho/LangSmith):
1. Check if `hooks.json` exists
2. If exists, parse JSON, check for our marker
3. If already installed, skip
4. If not, merge our hooks into existing arrays
5. If removing and arrays become empty, delete file

## Extension Registration
- `WebviewViewProvider` registered for view ID `agentArcade.officeView`.
- Contributes to `panel` viewContainer (bottom panel in Cursor).
- `package.json` must declare the view under `contributes.views.panel`.
- Two commands registered: `cursor-office.enableHooks` and `cursor-office.disableHooks`.

## Packaging
```bash
npm run build:webview && npm run compile && npx vsce package --no-dependencies
```
Produces `.vsix` file. Install with `code --install-extension agent-arcade-*.vsix` (or `cursor` CLI).

---
> Source: [ofershap/cursor-office](https://github.com/ofershap/cursor-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
