## lilipad

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lilipad is an Initial Liquidity Offering (ILO) and Launchpad platform built on the Aptos blockchain. It combines a Next.js 16 frontend, Cloudflare Workers backend, and Move smart contracts.

## Commands

### Frontend (Next.js)
```bash
npm run dev        # Development server on port 3000
npm run build      # Production build
npm run lint       # ESLint
```

### Worker (Cloudflare)
```bash
cd worker
npm run dev        # Remote dev (connects to deployed D1)
npm run dev:local  # Local dev with local D1
npm run deploy     # Deploy to Cloudflare
npm run db:init    # Initialize D1 database (remote)
npm run db:init:local  # Initialize D1 database (local)
```

### Smart Contracts (Move)
```bash
cd contract
aptos move compile
aptos move test
```

## Architecture

```
├── app/                    # Next.js App Router (pages, layouts, API routes)
├── components/             # React components (sidebar, navbar, ui/)
├── lib/                    # Client libraries for external services
├── constants/              # Network config, module address, sidebar links
├── worker/                 # Cloudflare Workers API (D1 SQLite database)
└── contract/sources/       # Move modules for on-chain logic
```

### Data Flow
1. **Frontend** → User interacts with React components using `useWallet()` hook
2. **API Proxy** → `app/api/projects/route.ts` forwards to Cloudflare Worker
3. **Worker** → CRUD operations on D1 database (project metadata storage)
4. **Blockchain** → Direct calls via client libraries using `@aptos-labs/ts-sdk`

### Key Client Libraries
- `lib/lilipadClient.ts` - Core Aptos interactions: token operations, locking, vesting, launchpad sales
- `lib/hyperionClient.ts` - Hyperion DEX integration: swaps, pools, liquidity positions
- `lib/dexscreenerClient.ts` - Price/chart data from DexScreener

### Move Contract Modules (contract/sources/)
- `launchpad.move` - Token sales with escrow, soft caps, and automatic vesting
- `token.move` - Fungible asset creation, minting, burning
- `locking.move` - Time-locked token deposits (fungible + LP tokens)
- `vesting.move` - Linear vesting streams with claimable releases
- `escrow.move` - Holds tokens/APT during sales until conditions met
- `events.move` - On-chain event emissions
- `constants.move` - Shared error codes and precision values
- `init.move` - Module initialization

### Launchpad Sale Flow
1. Owner creates sale via `create_sale()` with token, price, timing, soft cap
2. Owner deposits tokens to escrow via `deposit_sale_tokens()` → `escrowed = true`
3. Buyers purchase via `buy()` → APT escrowed, vesting stream created
4. After sale ends, owner withdraws APT via `withdraw_proceeds()`
5. Buyers claim vested tokens via `vesting::claim()`

## Tech Stack

- **Frontend**: Next.js 16, React 19, Tailwind CSS v4, shadcn/ui (Radix)
- **Blockchain**: Aptos (testnet default), @aptos-labs/ts-sdk, wallet-adapter-react
- **DEX**: Hyperion SDK (@hyperionxyz/sdk)
- **Backend**: Cloudflare Workers, D1 (SQLite)
- **Contracts**: Move language

## Environment Variables

```
NEXT_PUBLIC_WORKER_URL      # Cloudflare Worker URL (default: http://localhost:8787)
NEXT_PUBLIC_APP_NETWORK     # Aptos network (testnet/mainnet)
NEXT_PUBLIC_MODULE_ADDRESS  # Deployed Move module address
NEXT_PUBLIC_APTOS_API_KEY   # Optional Aptos RPC key
```

## Conventions

- All interactive pages use `"use client"` directive
- Components: PascalCase in code, files: kebab-case (`app-sidebar.tsx`)
- API responses: `{ success: boolean, data/error }`
- Wallet state via `useWallet()` hook from wallet adapter
- Transaction payloads built via `build*Payload()` functions, executed with `signAndSubmitTransaction()`
- BigInt used for all token amounts; use `parseTokenAmount()` / `formatTokenBalance()` for conversion
- Price precision: `PRICE_PRECISION = 1e9` for launchpad pricing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dipansrimany2006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
