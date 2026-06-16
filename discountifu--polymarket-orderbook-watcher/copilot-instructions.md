## polymarket-orderbook-watcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build (transpile JSX to dist/)
pnpm build

# Development mode (watch for changes)
pnpm dev

# Run tests (prettier + xo lint + ava tests)
pnpm test

# Run the CLI (requires Node.js 22+ for native WebSocket)
node dist/cli.js
```

## Architecture

Real-time orderbook viewer for Polymarket BTC Up/Down 15-minute markets. Automatically detects and switches to the current 15-minute window.

**Key technologies:**

- **Ink + React**: Terminal UI components in JSX
- **Babel**: Transpiles JSX from `source/` to `dist/` (preset: @babel/preset-react)
- **polymarket-websocket-client**: WebSocket client for CLOB orderbook data
- **meow**: CLI argument parsing
- **xo + prettier**: Linting and formatting

**Source structure:**

- `source/cli.js` - CLI entrypoint
- `source/app.js` - Main React/Ink component with orderbook display
- `source/utils.js` - Market data fetching and 15-min window utilities

**Data flow:**

1. Calculate current 15-min window timestamp (aligned to clock: 8:00, 8:15, 8:30, 8:45...)
2. Fetch market data from Gamma API (`gamma-api.polymarket.com/events/slug/{slug}`)
3. Extract `clobTokenIds` for Up/Down outcomes
4. Subscribe to orderbook via `ClobMarketClient` WebSocket
5. Display bids/asks in real-time, auto-switch at window boundary

---
> Source: [discountifu/polymarket-orderbook-watcher](https://github.com/discountifu/polymarket-orderbook-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
