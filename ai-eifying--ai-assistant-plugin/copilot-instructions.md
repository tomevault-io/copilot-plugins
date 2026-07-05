## ai-assistant-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Super Productivity plugin (manifest v1) that provides an AI chat panel for task management via OpenAI-compatible APIs. Single-file architecture — all CSS, HTML, and JS live inline in `index.html` (~1000 lines). No build system, no dependencies, no tests.

## File map

| File | Role |
|------|------|
| `manifest.json` | Plugin metadata, Super Productivity API permissions, hooks |
| `plugin.js` | Background script — minimal, just logs load |
| `index.html` | The entire plugin: CSS, settings modal, chat UI, tool definitions, AI loop, markdown renderer |
| `icon.svg` | Plugin icon |

## Key architecture (index.html)

**Runtime environment:** The Super Productivity host injects `PluginAPI` globally. All data operations go through it — there are no direct DB or filesystem calls.

**Data flow:**
1. `sendMessage()` appends user message + `buildContext()` snapshot to `messages[]`
2. `callOpenAI()` sends full conversation + tool definitions to the model
3. If the model returns `tool_calls`, `executeTool()` dispatches to matching `PluginAPI` calls, results appended as tool messages, loop continues (max 5 rounds)
4. Final text response rendered via custom `renderMarkdown()` and displayed

**Persistence:** Config and conversation (last 50 messages) are stored via `PluginAPI.loadSyncedData()` / `PluginAPI.persistDataSynced()` under keys `ai-assistant-config` and `ai-assistant-conversation`.

**Tool definitions** (`var tools = [...]`): 24 tools covering tasks (CRUD, bulk, subtasks, reorder, tags, planning, worklog), projects, tags, app context, and notifications. These are passed to OpenAI's function-calling API.

**Markdown rendering:** Custom regex-based renderer in `renderMarkdown()` — not a library. Handles headers, bold/italic, links, blockquotes, tables, code blocks, ordered/unordered lists. Code blocks/content are HTML-escaped.

**Reasoning model support:** `reasoning_content` from the API response is preserved in stored messages (v1.6.1).

## CI/CD

Push to `master` triggers `.github/workflows/release.yml`: reads version from `manifest.json`, creates a GitHub Release with a zipped plugin package (manifest + index.html + plugin.js + icon.svg + README). No-op if the version tag already exists.

**Release flow:** bump `version` in `manifest.json` → commit → push to master. CI handles the rest.

## Build / Package

```bash
cd ~/server/ai-assistant-plugin && python3 -c "
import zipfile
with zipfile.ZipFile('ai-assistant-plugin.zip', 'w', zipfile.ZIP_DEFLATED) as zf:
    for f in ['manifest.json', 'plugin.js', 'index.html', 'icon.svg']:
        zf.write(f, f)
"
```

## Version history convention

Commit messages follow the pattern `vX.Y.Z: short description`. Tags are `vX.Y.Z`.

---
> Source: [ai-eifying/ai-assistant-plugin](https://github.com/ai-eifying/ai-assistant-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
