## aiagentprojectmanagement

> |------|------|------|

# CLAUDE.md
| 角色 | 模型 | 場景 |
|------|------|------|
| 領導/審查 | Opus 4.6 | 架構決策、code review、計畫 |
| 執行/實作 | Sonnet 4.6 | 編碼、測試、建置修復 |
| 輕量 | Haiku 4.5 | 格式化、靜態分析 |

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Agent Project Management Workspace — a browser-based local workspace for running CLI agents and organizing work as draggable cards on an infinite canvas. Built with vanilla JS (no framework), Node.js/Express backend, WebSocket for real-time communication, and SQLite for persistence.

## Commands

```bash
npm install              # Install dependencies (requires Node.js 22+)
npm run dev              # Dev server with auto-reload (node --watch)
npm start                # Production start (default port 3000, auto-retries next port)
npm run test:unit        # Run all unit tests (node --test test/*.test.js)
npm run test:ui          # Headless browser smoke test
npm run test:all         # Unit + UI tests
npm run cards -- <cmd>   # CLI helper for card API (create, update, delete, list-cards, list-workspaces)
```

Single test file: `node --test test/some-file.test.js`

## Architecture

### Client-Server Flow

1. Browser connects via WebSocket, sends `init` with `clientId` (from localStorage)
2. Server hydrates state from SQLite, replies with `hydrate` message
3. All card mutations (create, move, resize, close) send `persist_state` over WebSocket
4. Terminal I/O flows through `node-pty` with echo detection (distinguishes user input echo from program output)
5. Multi-tab sync: `workspace-sync.js` broadcasts card CRUD to all sockets sharing a `clientId`

### Card System (Registry-Driven)

Cards are the core abstraction. The system is fully registry-driven — no hardcoded toolbar buttons or pane manager branching per type.

**To add a new card type:**
1. Create `public/js/my-card.js` extending `BaseCard`
2. Call `CardRegistry.register({ type, cardClass, buttonLabel, ... })` at the bottom
3. Add `<script src="/js/my-card.js"></script>` in `index.html` **before** `pane-manager.js`
4. If it needs API support, add its fields to `server/card-types.js` `CARD_FIELDS`

**Existing card types:** terminal (CLI), project, mission, agent-output, markdown

**Key BaseCard hooks to implement:**
- `getPersistData()` / `hydratePersistedData(data)` — card-specific data persistence
- `getUiPersistData()` / `hydrateUiState(data)` — UI state (color theme, etc.)
- `handleMessage(msg)` — WebSocket message handling
- `init()`, `fit()`, `dispose()` — lifecycle

### State Persistence

SQLite database at `data/web-terminal.sqlite` with two tables:
- `client_layouts` — panes_json, sections_json, active_pane_id per client
- `pane_buffers` — terminal output buffer + program-only buffer (echo-filtered) per pane

What persists: card layout/bounds, titles, card data, terminal buffers, color themes, workspace sections. What does NOT persist: live PTY processes.

### Key Subsystems

- **PaneManager** (`public/js/pane-manager.js`) — workspace orchestration: card lifecycle, selection, pan/zoom, group drag, section management, context menus, state hydration
- **BaseCard** (`public/js/base-card.js`) — drag/resize/focus/color themes via GestureManager, 8-directional resize handles
- **GestureManager** (`public/js/gesture-manager.js`) — unified pointer event abstraction (tap, drag, doubleTap, longPress)
- **PaneGeometry** (`public/js/pane-geometry.js`) — canvas coordinate math, bounds constraining
- **WorkspaceConfig** (`public/js/workspace-config.js`) — all tunable constants (zoom limits, grid snap, gesture thresholds)
- **OutputUtils** (`public/js/output-utils.js`) — ANSI stripping, agent phase detection, TUI chrome filtering

### WebSocket Message Types

Client → Server: `init`, `persist_state`, `create` (PTY), `input`, `resize`, `close`
Server → Client: `hydrate`, `output` (with echo/program origin tag), `error`, `exit`, `card_created`, `card_updated`, `card_deleted`

### Card API (HTTP)

`POST /api/cards`, `PATCH /api/cards/:paneId`, `DELETE /api/cards/:paneId` — for programmatic card manipulation. Only fields listed in `server/card-types.js` `CARD_FIELDS` are accepted (prevents arbitrary data injection).

## Testing Patterns

- Tests use Node.js built-in `node:test` and `node:assert/strict`
- Server modules use dependency injection (card service, state store, PTY manager accept mock implementations)
- State store uses `DatabaseSync` (synchronous SQLite) for deterministic testing
- UI smoke test (`scripts/test-ui-smoke.js`) uses Puppeteer; set `CHROME_BIN` for custom browser path

## Language

The project's UI, comments, and documentation are primarily in Traditional Chinese (zh-TW). CHANGELOG.md is written in Chinese. Code identifiers and API are in English.

---
> Source: [Yueh-H/AIagentProjectManagement](https://github.com/Yueh-H/AIagentProjectManagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
