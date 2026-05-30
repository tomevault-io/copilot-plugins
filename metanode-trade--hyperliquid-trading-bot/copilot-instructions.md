## hyperliquid-trading-bot

> This file provides guidance to AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Development Workflow

Before implementing anything, send me your plan of action for approval.
Start implementing things only after my explicit approval.

Follow test driven development.

## Package Management

Use **npm** for this repo:

- `npm install` — install dependencies
- `npm start` / `npm run dev` — run the bot (`tsx src/runBot.ts`)
- `npm run validate` — validate YAML (`tsx src/runBot.ts --validate`)
- `npm run typecheck` — `tsc` (no emit)

Requires **Node.js ≥ 20.19** (see `package.json` `engines`).

## Project Overview

TypeScript / Node.js grid trading bot for Hyperliquid: YAML configs under `bots/`, WebSocket mids, limit orders via `@nktkas/hyperliquid`, risk rules (drawdown, position size, optional SL/TP).

## Repository Structure

```
├── bots/                          # Bot configurations (YAML)
│   └── btc_conservative.yaml     # Sample conservative grid config
├── src/
│   ├── runBot.ts                 # CLI: validate, run, discover active config
│   ├── core/                     # Engine, config loader, keys, risk
│   │   ├── engine.ts
│   │   ├── enhancedConfig.ts
│   │   ├── keyManager.ts
│   │   └── riskManager.ts
│   ├── strategies/
│   │   ├── createStrategy.ts
│   │   └── grid/
│   │       └── basicGrid.ts
│   ├── exchanges/
│   │   ├── createAdapter.ts
│   │   └── hyperliquid/
│   │       ├── adapter.ts
│   │       └── marketData.ts
│   └── interfaces/
│       ├── strategy.ts
│       └── exchange.ts
└── .env                          # Local secrets (not in git)
```

## Configuration

YAML in `bots/` includes `active`, `exchange` (type, testnet), `account`, `grid`, `risk_management`, `monitoring`. See `README.md` for field-level detail.

## Running the System

```bash
# Auto-discover first active bots/*.yaml (or pass a path)
npm start
npx tsx src/runBot.ts bots/btc_conservative.yaml

# Validate only
npm run validate
npx tsx src/runBot.ts --validate
```

## Development Patterns

- **NO COMMENTS** in code unless explicitly requested
- Match existing module layout, naming, and strict TypeScript usage
- Prefer small, focused functions and clear interfaces (`src/interfaces/`)
- **Async I/O** — follow patterns in `marketData.ts` and the engine

### Error Handling

- Fail loudly with useful context; use structured logging (`pino`) at appropriate levels
- Clean shutdown on `SIGINT` / `SIGTERM` (see `runBot.ts` / engine)

### Testing / Safety

- Validate on **Hyperliquid testnet** before mainnet; use small size
- Run `npm run validate` after config changes
- Never commit private keys; use `.env` locally (see `.env.example` if present)

## Key Dependencies

- `@nktkas/hyperliquid` — Info / Exchange clients
- `viem` — wallet from private key
- `ws` — WebSocket client
- `js-yaml` — config parsing
- `pino` — logging
- `dotenv` — optional env loading

## Environment

Typical variables (see `README.md`):

- `HYPERLIQUID_TESTNET_PRIVATE_KEY` / `HYPERLIQUID_MAINNET_PRIVATE_KEY` / `HYPERLIQUID_PRIVATE_KEY`
- `HYPERLIQUID_TESTNET` — `true` / `false`

## Important Development Notes

- **Private key security**: never commit keys
- **Precision**: respect tick size / lot size from the exchange when placing orders
- **WebSocket reliability**: production runs need reconnect logic (follow existing `marketData` patterns)
- **Risk management**: use conservative settings for first tests

---
> Source: [metanode-trade/hyperliquid-trading-bot](https://github.com/metanode-trade/hyperliquid-trading-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
