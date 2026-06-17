## pharos-haven

> Vet ERC-20 tokens for honeypots, taxes, mintability, and proxy risks BEFORE swap or approve. The only Skill that answers on Pharos (chains 1672 mainnet, 688689 testnet) via native JSON-RPC, while still covering 60+ EVM chains via GoPlus. Returns a 0-100 risk score with safe/warn/block decision. Use whenever an agent encounters an unknown token contract.

# Pharos Haven

MCP stdio server with 4 tools for ERC-20 token security on Pharos and 60+ other EVM chains.

## Install

```
npx pharos-haven
```

Or add to Claude Desktop / Cursor config (see `/examples/claude-desktop-config.json`).

## Tools

- `check_token_goplus(chainId, address)` — GoPlus security scan on 60+ chains.
- `check_token_pharos_native(address, network)` — Pharos native ERC-20 metadata + deployment check.
- `aggregate_risk_score(chainId, address)` — Single entry point. 0-100 score + safe/warn/block decision.
- `explain_risk_verdict(chainId, address)` — Generates a human-friendly and AI-friendly natural language explanation and recommendation.

## Composability

Haven is built to be called by other Skills. See `examples/agent-loop.ts` for a 25-line safe-swap agent that consumes `aggregate_risk_score` before any approval. The same contract works as an OpenAI function-calling tool, an Anvita Flow Skill step, or a Cursor agent action.

## Example prompts for the agent

- "Is 0xdac17f958d2ee523a2206206994597c13d831ec7 on chain 1 safe to approve?"
- "Vet token 0x51e2A24742Db77604B881d6781Ee16B5b8fcBE29 on Pharos mainnet."
- "Aggregate risk score for CAKE on BSC: chainId 56, 0x0e09fabb73bd3ade0a17ecc321fd13a19e81ce82."

---
> Source: [Anand-0037/pharos-haven](https://github.com/Anand-0037/pharos-haven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
