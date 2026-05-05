## 00-system

> Core system context for Reineira Atlas — startup OS for ventures building on ReineiraOS


# Reineira Atlas — System Context

Reineira Atlas is the startup operating system for ventures building on ReineiraOS — open settlement
infrastructure for confidential programmable finance on Arbitrum.

## Protocol Stack (5 layers)

| Layer          | What It Does                                                 |
| -------------- | ------------------------------------------------------------ |
| Application    | End-user products — marketplaces, payroll, agent wallets     |
| Plugin         | Condition resolvers, insurance policies, custom verification |
| Protocol       | Escrow engine, coverage manager, pool factory, task executor |
| Infrastructure | Operator network, coordinator service, CCTP relay            |
| Settlement     | Arbitrum L2, Fhenix CoFHE, Circle CCTP V2                    |

## Ecosystem Repos

| Repo                 | Purpose                                                    |
| -------------------- | ---------------------------------------------------------- |
| **reineira-atlas**   | Startup OS — strategy, ops, growth, compliance, agents     |
| **reineira-code**    | Smart contracts — Solidity resolvers, policies, tests      |
| **platform-modules** | App template — backend (TypeScript), frontend (React 19)   |

## Shared Resources

All AI tools share the same knowledge base:
- **Docs (source of truth):** `.claude/docs/` — strategy, product, growth, intelligence, legal, execution
- **Data (append-only):** `.claude/data/` — decisions, metrics, signals
- **Domain rules:** `.cursor/rules/` — check descriptions to find relevant domain context

## Key Conventions

| Thing        | Convention                         | Example                                |
| ------------ | ---------------------------------- | -------------------------------------- |
| Doc files    | `SCREAMING_SNAKE.md` in domain dir | `.claude/docs/strategy/TOKENOMICS.md`  |
| Data files   | `YYYY-MM-DD.md` in stream dir      | `.claude/data/decisions/2026-03-20.md` |

## Data Flow Rules

1. **Before any recommendation:** Read `.claude/docs/intelligence/METRICS.md` for current numbers
2. **After any strategic change:** Log to `.claude/data/decisions/YYYY-MM-DD.md`
3. **After any metric update:** Update `.claude/data/metrics/YYYY-MM-DD.md`
4. Docs are source of truth. Only update docs when making real changes.
5. Data is append-only. Never edit past entries.

## Protocol Essentials

- **Chain:** Arbitrum (L2 on Ethereum)
- **Encryption:** Fhenix CoFHE (Fully Homomorphic Encryption on EVM)
- **Settlement:** Stablecoin-agnostic escrow (IFHERC20) — supports any wrapped stablecoin
- **Cross-chain:** Circle CCTP v2 (USDC cross-chain transfers)
- **Contracts:** Solidity ^0.8.24, UUPS upgradeable, ERC-7201 namespaced storage
- **Contract addresses:** Do not hardcode. Query from `CLAUDE.md` or MCP server.

## Plugin Interfaces

Builders extend the protocol through two interfaces:

**IConditionResolver** — controls when escrows release funds:
- `isConditionMet(escrowId) → bool` — view, called on every redeem
- `onConditionSet(escrowId, data)` — called once at escrow creation

**IUnderwriterPolicy** — evaluates risk and judges disputes (FHE-encrypted):
- `evaluateRisk(escrowId, proof) → euint64` — encrypted risk score 0-10000 bps
- `judge(coverageId, proof) → ebool` — encrypted dispute verdict

## Open Economy Roles

| Role              | What They Do                         | How They Earn                      |
| ----------------- | ------------------------------------ | ---------------------------------- |
| Policy Builders   | Write insurance policies             | Better risk models → more adoption |
| Pool Underwriters | Create pools, provide liquidity      | Net premiums = yield               |
| LP Stakers        | Deposit into pools                   | Share of premiums                  |
| Operators         | Relay cross-chain CCTP transactions  | 0.5% of bridged volume             |

## Tech Stack (from platform-modules)

| Layer             | Technology                                                   |
| ----------------- | ------------------------------------------------------------ |
| Frontend          | React 19 + TypeScript + Vite + Zustand + TanStack Router + TailwindCSS |
| Backend           | TypeScript + Clean Architecture (DB-agnostic, Vercel-ready)  |
| Wallet (primary)  | ZeroDev — ERC-4337 smart accounts, passkey auth              |
| Deploy            | Hardhat (contracts), Vercel (apps)                           |

## Platform Versioning

Current: Platform 0.1. Check `reineira.json` for repo version. Platform version is led by smart
contract interfaces — interface changes drive platform bumps.

## How Rules Work

This project has domain-specific rules in `.cursor/rules/`. Each rule's `description` field
contains keywords indicating when it should be activated. When your task matches a domain
(protocol, product, strategy, growth, ops, legal, pitch), the relevant rule provides
specialized guidance and checklists.

Available slash commands are in `.cursor/commands/`. Use them for structured workflows.

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
