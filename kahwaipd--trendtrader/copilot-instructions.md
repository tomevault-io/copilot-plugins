## trendtrader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TrendTrader is an ACP (Agent Cooperation Protocol) seller agent that provides cryptocurrency trading services on Hyperliquid. Buyers interact via ACP jobs to open/close positions, deposit/withdraw funds (via CCTP bridge), and set up TA-triggered conditional orders. Each buyer gets an isolated subaccount derived from the master wallet.

## Commands

```bash
npm run setup              # Initial setup - checks env, inits HL client
npm run seller:run         # Start the seller runtime (main process)
npm run dev                # Dev mode with auto-reload (tsx watch)
npm run test:ta            # Run technical analysis indicator tests
npm run offering:create    # Register offerings on ACP
npm run offering:delete    # Remove offerings from ACP
```

Utility scripts are run directly with `tsx scripts/<name>.ts` (e.g., `tsx scripts/check-order-status.ts`).

## Architecture

### Runtime Flow

ACP socket receives jobs → `seller/runtime/seller.ts` resolves the offering name from the job memo → loads the matching handler from `seller/offerings/<name>/handlers.ts` → validates requirements → executes → delivers result back to ACP.

On startup, the seller also initializes:
- **Order Monitor** (`src/orders/monitor.ts`) - WebSocket + 60s polling loop that evaluates TA signals on pending orders
- **Bridge Worker** (`src/bridge/worker.ts`) - 30s polling loop that progresses CCTP deposits through attestation → claim → Hyperliquid deposit

### Key Modules

| Module | Purpose |
|--------|---------|
| `seller/runtime/` | ACP socket connection, job acceptance/rejection/delivery orchestration |
| `seller/offerings/` | 17 offerings, each with `offering.json` (config) + `handlers.ts` (validate + execute) |
| `src/hyperliquid/` | Hyperliquid API wrapper - InfoClient, ExchangeClient, trading helpers, subaccount management |
| `src/indicators/` | TA engine: 50+ indicators via `technicalindicators` lib, signal evaluator supporting 9 signal types, Pine Script parser |
| `src/orders/` | PostgreSQL-backed pending order CRUD and WebSocket-based signal monitoring |
| `src/bridge/` | CCTP bridge (Base → Arbitrum → Hyperliquid), Across bridge integration, pending deposit tracking |
| `src/fees/` | Fee collection from subaccounts before trade execution |
| `src/users/` | Deterministic subaccount derivation from master key (`keccak256(masterKey + salt + index)`) |
| `src/audit/` | PostgreSQL audit logging and fee ledger |

### Signal System

The evaluator (`src/indicators/evaluator.ts`) handles 9 signal types defined in `src/indicators/signals.ts`: `threshold`, `crossover`, `band_touch`, `squeeze`, `divergence`, `pattern`, `multi` (AND/OR combinator), `range`, and `slope`. The `multi` type allows composing other signals recursively.

### Adding a New Offering

Create a directory under `seller/offerings/<name>/` with:
- `offering.json` - name, description, jobFee, requiredFunds, requirement schema
- `handlers.ts` - exports `validateRequirements()` and `executeJob()` (returning `ExecuteJobResult`)

### Database

PostgreSQL (configured via `DATABASE_URL` env var). Key tables: `pending_orders` (with JSONB `signal` column), `audit_log`, `bridge_deposits`, `fee_ledger`.

## Tech Stack

- **TypeScript 5.7** (ES2022 target, ESNext modules) - run via `tsx`, no build step needed
- **@nktkas/hyperliquid** + **hyperliquid** - Hyperliquid API clients
- **viem** - EVM primitives (accounts, contract calls, encoding)
- **technicalindicators** - 50+ TA indicators
- **pg** - PostgreSQL client
- **socket.io-client** - ACP WebSocket connection
- **ws** - Hyperliquid candle WebSocket

## Environment Variables

See `.env.example`. Key vars: `LITE_AGENT_API_KEY`, `HYPERLIQUID_PRIVATE_KEY`, `HYPERLIQUID_WALLET_ADDRESS`, `HYPERLIQUID_TESTNET`, `DATABASE_URL`, `MASTER_WALLET_PRIVATE_KEY` (for bridge), `FEE_WALLET_ADDRESS`.

Runtime config is also stored in `config.json` (API keys, wallet addresses, seller PID).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kahwaipd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kahwaipd)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
