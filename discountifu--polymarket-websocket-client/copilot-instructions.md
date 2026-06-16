## polymarket-websocket-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zero-dependency TypeScript WebSocket client for Polymarket CLOB and RTDS APIs. The library provides real-time connections to Polymarket's trading infrastructure via WebSocket channels.

## Build Commands

```bash
pnpm install          # Install dependencies
pnpm run build        # Full build (clean + ESM + CJS + rename)
pnpm run build:esm    # Build ES modules only
pnpm run build:cjs    # Build CommonJS only
pnpm run test         # Run tests via Node.js test runner
```

## Architecture

### Source Structure

```
src/
├── index.ts          # Public exports
├── types.ts          # All TypeScript type definitions
├── event-emitter.ts  # Typed event emitter utility
├── base-client.ts    # Abstract WebSocket client with reconnection/heartbeat
├── clob-client.ts    # CLOB Market and User channel clients
└── rtds-client.ts    # RTDS real-time data client
```

### Client Hierarchy

```
TypedEventEmitter<Events>
    └── BaseWebSocketClient (abstract)
            ├── ClobMarketClient  (public orderbook data)
            ├── ClobUserClient    (authenticated user data)
            └── RtdsClient        (real-time data streams)

ClobClient (unified wrapper for market + user)
```

### Key Design Patterns

1. **Event-based API**: All clients extend `TypedEventEmitter` for type-safe event handling
2. **Abstract Base Class**: `BaseWebSocketClient` handles connection lifecycle, reconnection with exponential backoff, and heartbeat
3. **Subscription Management**: Clients track subscriptions and automatically resubscribe on reconnection
4. **Callback Registration**: Methods like `onBook()`, `onTrade()` return unsubscribe functions

### Dual Module Output
- **ESM**: `dist/esm/` (`.js` files, NodeNext modules)
- **CJS**: `dist/cjs/` (`.cjs` files, CommonJS)

The `scripts/rename-cjs.mjs` post-build script renames CJS output from `.js` to `.cjs`.

## WebSocket Channels

### CLOB Market Channel (`wss://ws-subscriptions-clob.polymarket.com/ws/market`)
- Public orderbook data
- Subscribe by asset IDs (token IDs)
- Events: `book`, `price_change`, `tick_size_change`, `last_trade_price`

### CLOB User Channel (`wss://ws-subscriptions-clob.polymarket.com/ws/user`)
- Authenticated user data (requires apiKey/secret/passphrase)
- Subscribe by market IDs (condition IDs)
- Events: `trade`, `order`

### RTDS (`wss://ws-live-data.polymarket.com`)
- Real-time data streams
- Topics: `crypto_prices`, `crypto_prices_chainlink`, `comments`, `activity`
- 5-second heartbeat interval (vs 30s for CLOB)

## Implementation Notes

- `handleParsedMessage()` is the abstract method subclasses implement to process messages
- Subscriptions are stored in Sets/Maps and replayed on reconnection
- RTDS uses `{ type: 'PING' }` for heartbeat, CLOB uses simple `'ping'` string
- All callback methods return unsubscribe functions for cleanup

## Constraints

- Zero runtime dependencies - uses only native Node.js WebSocket
- Requires Node.js 22+ for native WebSocket support

---
> Source: [discountifu/polymarket-websocket-client](https://github.com/discountifu/polymarket-websocket-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
