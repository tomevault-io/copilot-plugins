## ytblog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YTBlog is a Chrome extension (Manifest V3) that transforms YouTube videos into scrollable blog articles. It extracts video captions, restructures them into clean prose via a local Ollama LLM, and presents the text as a research-backed readable article that replaces the video player in theater mode.

## Development

No build step — plain vanilla JavaScript, loaded directly by Chrome. To develop:

1. Open `chrome://extensions`, enable Developer Mode
2. Click "Load unpacked" and select this repo's root directory
3. Navigate to any YouTube video — a "Read" button appears if captions are available

For AI features, Ollama must be running locally (`ollama serve`) with a model pulled (default: `llama3.2`).

Testing is manual via `test.html` (opened as a local file) and the Playwright MCP browser tools.

## Architecture

### Two Content Script Worlds

The extension runs content scripts in two Chrome worlds on YouTube watch pages:

- **MAIN world** (`src/content/main-world.js`): Has access to YouTube's page JS context. Extracts caption tracks from `ytInitialPlayerResponse` / `ytplayer.config` and posts data to the ISOLATED world via `window.postMessage`.
- **ISOLATED world** (all other `src/content/` files): Cannot access page JS but can use `chrome.*` APIs. Runs the full pipeline: transcript fetch → parse → AI processing → blog rendering.

### Global Namespace Pattern

All ISOLATED world modules attach to `window.YTBlog`. There is no bundler or module system — scripts are loaded in dependency order via `manifest.json`'s content_scripts array. Order matters.

### Pipeline Flow

1. **Transcript extraction**: `main-world.js` → posts `YTBLOG_CAPTION_DATA` message
2. **Transcript fetch/parse**: `transcript/extractor.js` fetches json3 format, `parser.js` parses it
3. **AI processing**: Content script sends `PROCESS_TRANSCRIPT` message to service worker → service worker calls Ollama API with two-pass approach (structural scan + chunked restructuring)
4. **Rendering**: `content.js` orchestrates BlogRenderer + Highlighter

### Service Worker (`src/background/`)

ES modules (`import`/`export`). Handles:
- Ollama health checks and model verification (`ollama.js`)
- Two-pass LLM pipeline: structure analysis + chunked transcript restructuring (`service-worker.js`, `prompts.js`)
- Sends progress updates back to content script via `chrome.tabs.sendMessage`

### Blog Renderer (`src/content/blog-renderer.js`)

Renders structured data as a scrollable article with:
- 3 content-type styles only: paragraph (flow), key insight (impact), list (stack)
- No energy-based styling — research shows this creates visual noise
- Progress bar, TOC, section recaps, takeaways, reading time badge
- Copy as Markdown, timestamp links, chapter thumbnails

### Highlighter (`src/content/highlighter.js`)

Text highlighting system:
- Select text → floating toolbar → one-click highlight
- Stored by video ID + thought index in `chrome.storage.local`
- H key highlights closest paragraph, Z undoes last action
- Restored on revisit

### Player Replacer (`src/content/ui/player-replacer.js`)

Uses YouTube's theater mode for full-width reading area:
- Sets theater + full-bleed-player attributes on ytd-watch-flexy
- Creates stage inside #full-bleed-container
- Falls back to inserting before player if theater elements missing

### Keyboard Shortcuts (active during reading)

T/Esc=close, H=highlight paragraph, Z=undo highlight

## Key Data Structures

The AI produces this shape, consumed by BlogRenderer:

```json
{
  "sections": [{
    "title": "string",
    "recap": "string",
    "thoughts": [{
      "text": "string",
      "emphasis": ["word1"],
      "mode": "flow|impact|stack",
      "energy": "calm_intro|explanation|...",
      "complexity": 0.0-1.0
    }]
  }],
  "takeaways": ["string"]
}
```

Note: `energy` and `complexity` fields are present in the AI output but NOT used for visual styling. Only `mode` and `emphasis` drive the 3-style visual system.

## Design Principles

- **3 visual styles only**: Regular paragraph, key insight (bold + accent bg), list. Based on coherence principle research (d=0.86).
- **No pull quotes**: NNG eyetracking shows they cause readers to abandon linear reading.
- **Theater mode layout**: Full viewport width, not constrained scroll box.
- **Self-paced scrolling**: Native scroll, user controls pace (research: self-paced > system-paced).
- **Typography**: Georgia 19px, 1.7 line height, 680px max width (~65 chars).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IFAKA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IFAKA)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
