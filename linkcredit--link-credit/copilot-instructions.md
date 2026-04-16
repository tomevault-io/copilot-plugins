## link-credit

> AI-powered privacy credit scoring for low-collateral DeFi lending. Built for the [Chainlink Hackathon](https://chain.link/hackathon) (Privacy + DeFi tracks).

# CLAUDE.md — Link Credit (Houston)

## Project Overview

AI-powered privacy credit scoring for low-collateral DeFi lending. Built for the [Chainlink Hackathon](https://chain.link/hackathon) (Privacy + DeFi tracks).

**Core flow**: User authorizes bank data (Plaid Link) → CRE Workflow exchanges token + fetches bank data + AI scores credit (0-100) via Confidential HTTP → Score written on-chain → Lending pool offers lower collateral ratio.

## Architecture

```
L3: React DApp (frontend) — wallet connect, Plaid Link auth, view score, borrow
L2: CRE Workflow (TypeScript) — token exchange + Confidential HTTP (Plaid data) → AI scoring → on-chain write
L1.5: Serverless Function — Plaid Link token creation only (Cloudflare Worker)
L1: Solidity Contracts (Sepolia) — CreditOracle.sol + Fork Aave v3 (modified GenericLogic)
Testing: Tenderly Virtual TestNet (Sepolia mirror RPC, same chain id/address set)
```

## Tech Stack

- **Contracts**: Solidity, Foundry (deploy on Sepolia), Fork of `aave-dao/aave-v3-origin` (v3.6)
- **CRE Workflow**: TypeScript, `@chainlink/cre-sdk`, compiled to WASM
- **Serverless**: Cloudflare Worker (Plaid Link token creation only, ~10 lines)
- **Frontend**: React + wagmi/viem + TailwindCSS (optional Tenderly Virtual TestNet RPC)
- **Package manager**: bun (preferred), pnpm as fallback

## Key CRE SDK Constraints

- Workflow callbacks are **stateless** — each trigger starts from zero
- Runs in **QuickJS** engine, NOT full Node.js — no `node:crypto` or built-in modules
- AI inference must be done via **external HTTP API** calls
- Multi-node execution with **BFT consensus** (use `cre.consensusMedianAggregation`)
- Confidential HTTP: API keys stored in Vault DON, data processed in TEE
- **No HTTP response** — CRE writes results on-chain, cannot return data to frontend
- Max 5 HTTP requests per execution — budget carefully (Plaid calls + AI scoring)

## Contracts

- `CreditOracle.sol` — stores per-user credit scores (0-10000 bps), computes LTV boost
- **Fork Aave v3** (`aave-v3-origin`) — modified `GenericLogic.calculateUserAccountData()` to read credit boost from CreditOracle, ~10 lines core change
- Only the CRE workflow address can write to the oracle
- See `aave-fork.md` for detailed Aave fork architecture

## Build & Dev Commands

```bash
# Install all workspace dependencies
bun install

# Sync git submodules (required for contracts libs like aave-v3-origin/forge-std)
git submodule update --init --recursive

# Run dev servers
bun run dev:frontend   # React DApp
bun run dev:fork       # API + frontend using Tenderly Sepolia Virtual TestNet

# Build all
bun run build

# Contracts (Foundry — run from packages/contracts/)
cd packages/contracts && forge build && forge test
```

## Project Structure (Bun Monorepo)

```
houston/
├── package.json            # Root — workspaces: ["packages/*"]
├── tsconfig.json           # Shared TS config
├── packages/
│   ├── contracts/          # @link-credit/contracts — Foundry (Solidity)
│   ├── workflow/           # @link-credit/workflow — CRE workflow (TS → WASM)
│   ├── api/                # @link-credit/api — Serverless (Cloudflare Worker, Plaid Link token only)
│   └── frontend/           # @link-credit/frontend — React DApp (Vite)
├── context.md              # Original architecture plan (Chinese)
├── aave-fork.md            # Aave v3 fork technical details (Chinese)
└── CLAUDE.md               # This file
```

## Conventions

- Use **TypeScript** everywhere (no plain JS)
- Use **bun** as package manager
- Solidity: use Foundry (forge), not Hardhat
- Keep contracts minimal — this is a hackathon MVP
- Mock data is acceptable — focus on architecture and CRE integration
- Language: code and comments in English, context.md is in Chinese

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LinkCredit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
