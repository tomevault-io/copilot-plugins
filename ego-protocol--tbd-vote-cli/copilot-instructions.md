## tbd-vote-cli

> Trade on TBD, a Solana-based prediction market for human opinions, using the @tbd-vote/cli npm package. TRIGGER when: the user mentions tbd.vote, TBD, or @tbd-vote/cli; asks to bet, trade, or wager on an opinion or poll on TBD; or wants to list or browse opinion-market campaigns on Solana. DO NOT TRIGGER for Polymarket, Kalshi, Manifold, or other event-outcome prediction markets, or for general Solana DeFi unrelated to opinion markets.


# TBD — Solana Prediction Market for Human Opinions

[TBD](https://www.tbd.vote) is a Solana-based prediction market where users wager on human opinions — what people think, prefer, and predict — rather than purely objective event outcomes.

**The complete agent guide, CLI reference, autonomous loop instructions, and raw HTTP fallback live at https://www.tbd.vote/agents/AGENTS.md — read that file as the source of truth for anything not covered below.**

## Setup

```bash
npm install -g @tbd-vote/cli
tbd-vote login
```

The user must provide their own API key. Never guess or generate credentials.

## Quick command reference

| Command | Purpose |
|---|---|
| `tbd-vote login` | Authenticate with an API key |
| `tbd-vote campaigns list --json` | Browse open opinion-poll campaigns (always use `--json` for parsing) |
| `tbd-vote bet <campaign-id> <option-id>` | Place a bet on a specific option |

## Guidelines

- **Always parse `--json` output** rather than scraping the human-readable format.
- **Always confirm bets with the user** before executing them. Bets are real on-chain transactions and irreversible.
- **Treat opinion markets differently from event markets**: outcomes resolve based on what people believe or vote, not on objective real-world events.
- **For autonomous loops, deeper integration, or anything not covered here, read https://www.tbd.vote/agents/AGENTS.md** — it documents the full autonomous-loop pattern, error handling, and HTTP fallback.

## Links

- Website: https://www.tbd.vote
- AGENTS.md (source of truth): https://www.tbd.vote/agents/AGENTS.md
- CLI repo: https://github.com/ego-protocol/tbd-vote-cli
- npm: https://www.npmjs.com/package/@tbd-vote/cli

---
> Source: [ego-protocol/tbd-vote-cli](https://github.com/ego-protocol/tbd-vote-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
