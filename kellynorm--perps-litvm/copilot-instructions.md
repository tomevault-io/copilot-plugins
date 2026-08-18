## perps-litvm

> This file is the project constitution. Read it at the start of every session, then read `TASK.md` for the current task. Work only within the scope of the task marked **CURRENT**.

# CLAUDE.md — Perps DEX on LitVM

This file is the project constitution. Read it at the start of every session, then read `TASK.md` for the current task. Work only within the scope of the task marked **CURRENT**.

## What we are building
A GMX-style, pool-based perpetual (leverage) DEX on LitVM.

- Network: LitVM **LiteForge** testnet
- Chain ID: **4441**
- Gas token: **zkLTC** (free from the Caldera faucet)
- Execution stack: Arbitrum Nitro (EVM-equivalent) — standard Solidity/Foundry tooling applies

This is a durable, mainnet-grade DeFi product, **not** a testnet farming toy. The leverage-venue lane is currently open on LitVM; the goal is to occupy it with something that survives to mainnet and the TGE.

## Architecture (target)
- **Liquidity pool (GLP-style):** LPs deposit collateral, mint an LP token, and act as the counterparty to all traders.
- **Positions:** open/close long & short with leverage; collateral + P&L accounted against an oracle mark price.
- **Funding rate:** periodic funding keeps long/short open interest balanced.
- **Liquidations:** permissionless and bounty-incentivized — third-party liquidators run themselves, so we pay for no keeper farm.
- **Execution:** two-step deferred (request -> execute next block) with the price relayed on-chain at execution. This is the MEV / front-running protection.

## Oracle
- Provider: **RedStone Pull Model** (on-demand signed prices injected into tx calldata; verified on-chain).
- Why pull: works on LitVM without RedStone pre-deploying feed contracts; purpose-built for perps; gas-cheap.
- Testnet data service: `redstone-main-demo` (free). Solidity consumer base: `MainDemoConsumerBase`. Feed ids passed as `bytes32` (e.g. "BTC", "ETH").
- JS side injects the payload via `DataServiceWrapper`.
- Rules: always validate signatures AND timestamps; enforce a tight staleness window for any price used in liquidation or P&L; never settle a fund-moving action on a stale or unverified price.
- LitOracle (LitVM's native push oracle) may be used as a fallback or for spot/display only — **never** as the sole liquidation price.

## Stack
- Contracts: Solidity + Foundry (forge / cast / anvil)
- Oracle libs: `@redstone-finance/evm-connector`, `@redstone-finance/sdk`
- Frontend: Next.js + Tailwind + shadcn/ui, hosted on Cloudflare Pages (free)
- Keeper/relayer: lightweight Node service on a free always-on VM (e.g. Oracle Cloud free tier)
- CI: GitHub Actions

## Discipline rules (non-negotiable)
1. One feature per branch, one branch per PR. No mixing concerns.
2. Tests are mandatory for every PR. No merge without passing `forge test`.
3. Plan before large changes. State the plan, get a go-ahead, then implement.
4. Squash-merge to `main`. Never force-push a feature branch.
5. Never leave stray uncommitted files — treat them as entanglement risk.
6. After each merge, deploy and validate on testnet (chain 4441) before starting the next branch.
7. Run `forge fmt` before every commit.

## Money-path & safety rules (this contract holds pooled funds)
- Any code touching collateral, P&L, pricing, funding, or liquidation is **HIGH RISK**. For these: write the reasoning first, then the tests, then the implementation.
- Apply checks-effects-interactions and reentrancy guards on every fund-moving function.
- Prefer `immutable` / `constant`; minimize admin powers; document every privileged function.
- Never ship to mainnet unaudited. No real funds until an audit (target: LitVM builder-program audit credits).
- Flag any rule conflict or money-path risk explicitly **before** writing code, not after.
- No secrets in the repo. Keys and RPC URLs live in `.env` (gitignored).

## How Claude Code should operate here
- Start by reading this file, then `TASK.md`. Identify the Current Task.
- Implement only the Current Task's scope. Do not touch out-of-scope files.
- Write/extend tests. Run `forge test` and `forge fmt`. Report results.
- If a change would touch the money path or conflict with a rule, stop and flag it first.
- Do not modify any already-deployed contract or its ABI without an explicit, separate instruction.

---
> Source: [KellyNorm/perps-litvm](https://github.com/KellyNorm/perps-litvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
