## alphaloop

> > **For AI agents reading this repo**: this is a *skill suite*, not an application. It defines a human-agent collaboration operating model for investment research and bundles the six sub-skills it orchestrates.

# AlphaLoop — Agent Instructions

> **For AI agents reading this repo**: this is a *skill suite*, not an application. It defines a human-agent collaboration operating model for investment research and bundles the six sub-skills it orchestrates.

## What you are looking at

AlphaLoop is an **operating model**: the trader sets direction, the agent gathers and verifies evidence, and the human always keeps the decision. The full specification lives in [`SKILL.md`](SKILL.md). **Read it before doing anything else.**

This `AGENTS.md` exists so that agents following the [`AGENTS.md` standard](https://github.com/openai/codex/blob/main/AGENTS.md) (OpenAI Codex, OpenCode, Hermes, OpenClaw, Aider, etc.) can discover and load the suite consistently.

## Quick install (any agent)

```bash
git clone https://github.com/realnaka/alphaloop.git
cd alphaloop
./install.sh
```

The installer auto-detects which agents are present and symlinks the orchestrator plus every bundled sub-skill into each agent's skills directory. It also initializes an empty wiki at `${OPENORDER_HOME:-$HOME/openorder}` (templates only, no real data).

## How to use as an agent

Whenever a conversation enters an investment-research context (a ticker, a company, an industry, a forwarded research note or screenshot, a thesis to verify, a stock to pick, a trade to log):

1. Treat any logic/claim the human hands you as a **hypothesis (C-grade)**, not a fact. Verify before trusting. → see [`skills/claim-verification/SKILL.md`](skills/claim-verification/SKILL.md)
2. **First action** when a ticker/industry/framework appears: read `${OPENORDER_HOME}/INDEX.md` → relevant files. → [`skills/openorder/SKILL.md`](skills/openorder/SKILL.md)
3. Always pull **real-time** prices with a timestamp + source; never use stale data. → [`skills/stock-data-fetch/SKILL.md`](skills/stock-data-fetch/SKILL.md)
4. For raw materials / chokepoint / export-control topics, run the five-layer funnel. → [`skills/strategic-materials/SKILL.md`](skills/strategic-materials/SKILL.md)
5. Hand back a **verification matrix + stock list + risks**; never pull the trigger for the user. The decision is always the human's.
6. When a trade happens, log it with its **source framework** to close the feedback loop. → [`skills/trade-journal/SKILL.md`](skills/trade-journal/SKILL.md)
7. If any tool fails (403/404/can't fetch), escalate the tool, not the user. → [`skills/agent-tool-escalation/SKILL.md`](skills/agent-tool-escalation/SKILL.md)

## Key files in this repo

| File | Purpose |
|---|---|
| [`SKILL.md`](SKILL.md) | Orchestrator spec — division of labor, routing table, three iron rules (source of truth) |
| [`AGENTS.md`](AGENTS.md) | This file (mirror for `AGENTS.md`-aware agents) |
| [`README.md`](README.md) | Human-facing pitch (Chinese) and quick start |
| [`install.sh`](install.sh) | Cross-agent installer |
| [`skills/`](skills/) | The six bundled sub-skills |
| [`docs/workflow.md`](docs/workflow.md) | The collaboration workflow in depth |
| [`docs/case-study.md`](docs/case-study.md) | A real, sanitized end-to-end walkthrough |
| [`docs/customize.md`](docs/customize.md) | How to fork the model for other domains |

## What this repo does NOT contain

- **No real research data**: only the methodology + templates. Your wiki content (company profiles, positions, P&L) lives at `${OPENORDER_HOME}` on your machine and is **never** committed here.
- **No secrets**: no API keys, no private spreadsheet tokens. Keys go in environment variables; the suite ships free, no-key price fallbacks.
- **No telemetry / network calls of its own**: pure markdown + your agent. Nothing phones home.

## Boundary

AlphaLoop gives the human **input, not a trigger**. Position sizing, timing, and the buy/sell decision always stay with the trader. The agent surfaces evidence, candidate names, and risks — and explicitly flags the bear case.

---

**Read [`SKILL.md`](SKILL.md) next.**

---
> Source: [realnaka/alphaloop](https://github.com/realnaka/alphaloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
