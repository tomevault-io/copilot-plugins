---
name: autonomous-agent
description: CreditNexus x402 agent. Use when the user wants stock predictions, backtests, bank linking, or agent/borrower scores. Payment-protected MCP tools (run_prediction, run_backtest, link_bank_account, get_agent_reputation_score, get_borrower_score, and by-email variants) with x402 flow (Aptos + Base). Agent handles 402 → pay → retry autonomously. Supports wallet attestation (signing) for onboarding. Use when this capability is needed.
metadata:
  author: openclaw
---

# CreditNexus x402 Agent Skill

Autonomous agent that **creates, funds, and uses** Aptos and EVM wallets, then calls x402-paid MCP tools (stock prediction, backtest, link_bank_account, agent/borrower scores). Handles payment flow (402 → pay → retry with `payment_payload`) with Aptos and Base. Supports **wallet attestation** (signing) for onboarding. Use when the user wants wallet setup, funding, balances, predictions, backtests, bank linking, scores, or token operations.

## Installation

Clone and install from the repository root:

```bash
git clone https://github.com/FinTechTonic/autonomous-agent.git && cd autonomous-agent
npm install
```

Set `MCP_SERVER_URL` to your x402 MCP server (e.g. `https://borrower.replit.app` or `https://arnstein.ch`). Copy `.env.example` to `.env` and configure LLM and wallet paths.

---

## Tasks you can do

### Wallets – create and manage

- **Create Aptos wallet** – `create_aptos_wallet` (optionally `network: "testnet"` or `"mainnet"`). Agent can have multiple Aptos wallets.
- **Create EVM wallet** – `create_evm_wallet` (optionally `network: "testnet"` or `"mainnet"`). Agent can have multiple EVM wallets.
- **List wallet addresses** – `get_wallet_addresses` returns all Aptos and EVM addresses (with network tags) for whitelisting and funding.

### Fund wallets

- **Fund Aptos wallet** – `credit_aptos_wallet`: on devnet uses programmatic faucet; on testnet returns instructions and Aptos faucet URL. Needed for `run_prediction` and `run_backtest` (~6¢ USDC).
- **Fund EVM wallet** – `fund_evm_wallet`: returns address and instructions (Base Sepolia faucet, etc.). Needed for `link_bank_account` (~$3.65 on Base).

User must **whitelist** every agent address at the onboarding flow (e.g. `http://localhost:4024/flow.html` or your MCP server’s flow) before paid tools succeed.

### Check balances

- **Aptos balance** – `balance_aptos` (USDC for the agent wallet).
- **EVM balance** – `balance_evm` (native token on a chain: base, baseSepolia, ethereum, etc.).

### Paid MCP tools (x402)

- **Stock prediction** – `run_prediction` (symbol, horizon in days). Cost ~6¢ (Aptos).
- **Backtest** – `run_backtest` (trading strategy). Cost ~6¢ (Aptos).
- **Link bank account** – `link_bank_account` (CornerStone/Plaid bank link token). Cost ~5¢ configurable (EVM/Base).
- **Scores** – `get_agent_reputation_score`, `get_borrower_score` (by wallet); `get_agent_reputation_score_by_email`, `get_borrower_score_by_email` (when SCORE_BY_EMAIL_ENABLED). x402 or lender credits.

The agent handles 402 Payment Required automatically: verify → settle → retry with payment signature.

### CLI (from repo root)

| Task | Command |
|------|--------|
| Generate Aptos wallet | `npm run setup:aptos` |
| Generate EVM wallet | `npm run setup` |
| Show addresses for whitelist | `npm run addresses` |
| Credit Aptos (devnet) | `npm run credit:aptos` (set `APTOS_FAUCET_NETWORK=devnet`) |
| EVM balance | `npm run balance -- <chain>` |
| Transfer ETH/tokens | `npm run transfer -- <chain> <to> <amount> [tokenAddress]` |
| Swap tokens (Odos) | `npm run swap -- <chain> <fromToken> <toToken> <amount>` |
| Run agent | `npx cornerstone-agent "Run a 30-day prediction for AAPL"` or `npx cornerstone-agent` (interactive); from repo `npm run agent -- "..."` |
| Attest Aptos wallet | `npm run attest:aptos` or `npx cornerstone-agent-attest-aptos` (output → POST /attest/aptos) |
| Attest EVM wallet | `npm run attest:evm` or `npx cornerstone-agent-attest-evm` (output → POST /attest/evm) |

---

## When to use this skill

Use when the user wants to:

- Create or use **Aptos** or **EVM** wallets (testnet or mainnet).
- **Fund** agent wallets (faucet instructions or programmatic credit).
- **Check** Aptos or EVM balances.
- Run **stock predictions** or **backtests** (paid via Aptos).
- **Link a bank account** (paid via Base; `link_bank_account`).
- **Get agent/borrower scores** (by wallet or by email when enabled).
- **Sign wallet attestations** for onboarding (attest:aptos, attest:evm).
- **Transfer** or **swap** tokens from the agent wallet (via CLI or context).

---

## Setup

1. **Install:** From repo root: `npm install`. Copy `.env.example` to `.env`.
2. **Configure:** Set `MCP_SERVER_URL`, `X402_FACILITATOR_URL`, `HUGGINGFACE_API_KEY` (or `HF_TOKEN`), `LLM_MODEL`, and wallet paths (`APTOS_WALLET_PATH`, `EVM_WALLET_PATH` or `EVM_PRIVATE_KEY`).
3. **Wallets:** Create via agent tools (`create_aptos_wallet`, `create_evm_wallet`) or CLI (`node src/setup-aptos.js`, `node src/setup.js`). Fund and whitelist all addresses at the MCP server’s flow (e.g. `/flow.html`).

---

## Run the agent

From the **repository root** (where `package.json` and `src/` live):

```bash
npx cornerstone-agent "Create an Aptos wallet, then run a 30-day prediction for AAPL"
# Or interactive
npx cornerstone-agent
# Or from repo: npm run agent -- "..." or node src/run-agent.js "..."
```

**Source:** [FinTechTonic/autonomous-agent](https://github.com/FinTechTonic/autonomous-agent)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
