## reineira-atlas

> Startup operating system for ventures building on ReineiraOS — open settlement infrastructure for

# Reineira Atlas

Startup operating system for ventures building on ReineiraOS — open settlement infrastructure for
confidential programmable finance on Arbitrum with Fhenix FHE.

## Domains

| Domain   | What It Covers                                             |
| -------- | ---------------------------------------------------------- |
| Protocol | Condition resolvers, insurance policies, FHE, Solidity     |
| Product  | Frontend (React 19), backend (TypeScript), SDK integration |
| Strategy | Business model, pricing, tokenomics, roadmap               |
| Growth   | Content, community, partnerships, developer relations      |
| Ops      | Sprint planning, weekly reviews, metrics                   |
| Legal    | MiCA, AML/KYC, GDPR, smart contract audits                |
| Pitch    | Investor readiness, deck prep, fund introductions          |

## Shared Resources

Both Claude Code and Cursor read from the same knowledge base:

- **Docs:** `.claude/docs/` — source-of-truth documents (strategy, product, growth, legal, etc.)
- **Data:** `.claude/data/` — append-only streams (decisions, metrics, signals)

## Tool-Specific Setup

| Tool        | Config Directory  | Entry Point                    |
| ----------- | ----------------- | ------------------------------ |
| Claude Code | `.claude/`        | `CLAUDE.md` + `.claude/SYSTEM.md` |
| Cursor      | `.cursor/`        | `.cursor/rules/00-system.mdc`  |

## Key Commands

| Command              | What It Does                                   |
| -------------------- | ---------------------------------------------- |
| `/bootstrap`         | Full venture setup from brief                  |
| `/resolver`          | Design a condition resolver                    |
| `/strategy`          | Strategic analysis and recommendation          |
| `/integrate`         | Wire protocol to app end-to-end                |
| `/weekly-plan`       | Weekly sprint review and planning              |
| `/pitch-prep`        | Investor readiness assessment                  |
| `/content`           | Create tutorials, blogs, social threads        |
| `/compliance`        | Crypto regulatory compliance review            |

## Protocol Context

- **Chain:** Arbitrum L2
- **Encryption:** Fhenix CoFHE (FHE on EVM)
- **Settlement:** Stablecoin-agnostic escrow (IFHERC20)
- **Cross-chain:** Circle CCTP v2
- **Contracts:** Solidity ^0.8.24, UUPS upgradeable
- **Contract addresses:** See `CLAUDE.md` or query MCP server

---
> Source: [ReineiraOS/reineira-atlas](https://github.com/ReineiraOS/reineira-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
