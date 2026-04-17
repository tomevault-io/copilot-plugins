## agent-verification-network

> > This file is read by Claude Code at the start of every session.

# CLAUDE.md — Agent Labor Market

> This file is read by Claude Code at the start of every session.
> It contains everything needed to understand and continue building this project.

## ⚠️ Check Memory First

Before starting work, **read the memory files** for the latest project state:
- `project_terminology_redesign.md` — **DECISIONS LOCKED**: Client/Worker/Manager. Job not task. /jobs not /verify. Ready for implementation.
- `project_agent_labor_market.md` — Core vision, what's built, community direction.
- `feedback_terminology_decisions.md` — Job not task, /jobs not /verify, dynamic spot checks, AVNC in x402.
- `feedback_no_push_main.md` — Use feature branches, never push to main.
- `feedback_no_gh_cli.md` — No gh CLI. Push branch and give URL for PRs.
- `feedback_production_focus.md` — Build for production. Three-layer architecture is sacred.

**Status (March 29, 2026):**
- RENAME COMPLETE: miner→worker, validator→manager in all files, endpoints, imports, UI
- Branch: `feature/enforce-role-separation` (includes role separation + rename)
- PR #44 still open on GitHub — merge before pushing this branch
- Still TODO: README rewrite, skill.md rewrite, agent.json update, /verify→/jobs endpoint rename

**Full system design document:** `~/Desktop/docs/agent_labor_market.html`
**Plan file:** `~/.claude/plans/lexical-greeting-babbage.md`

---

## What This Project Is

An open **job-agnostic marketplace for AI agents**. Clients post jobs, workers do them, managers check quality using spot checks (jobs with known answers). Everything on Base Mainnet.

**Three roles:**
- **Client** — posts jobs, pays for results (API key, x402, or on-chain escrow)
- **Worker** (currently "miner" in code) — does the work, earns 85% of payment
- **Manager** (currently "validator" in code) — routes jobs, runs spot checks, scores quality, earns 15% fee

**The core loop:**
1. Client submits a job + intent ("what should this code do?")
2. Manager routes to all registered workers + mixes in spot checks
3. Workers independently analyze (AST parsing + pattern detection + LLM intent matching)
4. Manager scores everyone: spot check accuracy + consensus + format + speed
5. Best result returned to client. ALL workers get rated (even losers).
6. Ratings recorded on-chain via AgentScorer.sol

**Three verticals live:** Code verification, text review, image validation. Protocol is job-agnostic — any vertical where ground truth can be constructed.

**Four payment methods:** API key credits (free tier), x402 micropayments (ETH/USDC/AVNC), on-chain escrow (AVNC), faucet (free 20 AVNC).

## Origin

Started as the **Agent Orchestration Protocol** — a system for coordinating multiple AI agents. Self-verification is self-referential. This project externalizes verification to a competitive market.

First version was a **Bittensor subnet** (~3,200 lines). Core verification logic was chain-agnostic, so we extracted it and rebuilt for Ethereum/Base.

---

## Terminology Rename (IN PROGRESS)

The codebase currently uses crypto terms. We are renaming to universal terms:

| Concept | Current (code) | Target | On-chain |
|---------|---------------|--------|----------|
| Does work | miner | **worker** | `provider` |
| Checks quality | validator | **manager** | `evaluator` |
| The work | task | **job** | — |
| Quality test | honeypot | **spot check** | — |
| Quality metric | score | **rating** | — |
| Main endpoint | /verify | **/jobs** | — |
| Browse work | marketplace | **job board** | — |

### Files to rename:
```
agent_market/miner/        → agent_market/worker/
agent_market/validator/    → agent_market/manager/
agents/miner_agent.py      → agents/worker_agent.py
agents/validator_agent.py  → agents/manager_agent.py
agents/miner_strategies.py → agents/worker_strategies.py
web/app/become-a-miner/   → web/app/become-a-worker/
web/app/become-a-validator/ → web/app/become-a-manager/
Dockerfile.miner           → Dockerfile.worker
```

### Endpoints (done / still TODO):
```
POST /register-miner     → POST /register-worker  ✅ DONE
POST /register-validator → POST /register-manager  ✅ DONE
POST /verify             → POST /jobs              ❌ TODO (next phase)
```

---

## On-Chain Artifacts

| Artifact | Chain | Address |
|----------|-------|---------|
| AgenticCommerceV2 | Base Mainnet | `0xA501a028F6C1d717009B65617540610aF25F02e7` |
| AgentScorer | Base Mainnet | `0x4e588353E7f247782A6109Fff3BA284a20D88c0F` |
| MinerRegistry | Base Mainnet | `0xf80DA8B7687685Bc96bf521085Ac1C0eea64bbDd` |
| ProtocolCredits (AVNC) | Base Mainnet | `0x6f1F2C3DB90DFc2956A7Ba1CB8bFf31420B4cc8F` |
| ERC-8004 Agent ID | Base Mainnet | #34655 on official registry |
| EigenCompute TEE | Intel TDX | App `0x7Fc30484...` at 34.142.184.34:8000 |

---

## File Map (POST-RENAME)

```
agent_market/
├── protocol.py              # Pydantic data contracts
├── chain.py                 # Web3.py → AgentScorer.sol
├── erc8004.py               # ERC-8004 Identity + Reputation
├── commerce.py              # On-chain job lifecycle (AgenticCommerceV2)
├── registry.py              # On-chain agent registry client
├── keys.py                  # API key management (Supabase)
├── token.py                 # AVNC token client
├── x402.py                  # x402 payment protocol (ETH/USDC/AVNC)
├── storage.py               # Filecoin/IPFS storage
├── logger.py                # Structured event logger
├── worker/                  # (was miner/)
│   ├── analyzer.py          # Code analysis (AST + patterns + LLM)
│   ├── text_analyzer.py     # Text review (grammar + tone + accuracy)
│   ├── image_analyzer.py    # Image validation (format + dimensions + Venice AI)
│   └── forward.py           # Worker entry point
├── manager/                 # (was validator/)
│   ├── honeypot.py          # Spot check generator (12 bug + 2 clean templates)
│   ├── image_honeypot.py    # Image spot checks
│   ├── scorer.py            # Rating formula (spot check + consensus + format + speed)
│   └── forward.py           # Manager loop (routes jobs, scores workers)
└── api/
    └── server.py            # FastAPI: all endpoints (1574 lines)

agents/
├── worker_agent.py          # (was miner_agent.py)
├── manager_agent.py         # (was validator_agent.py)
└── worker_strategies.py     # (was miner_strategies.py)

contracts/                   # Deployed, no rename needed
├── AgentScorer.sol
├── AgenticCommerceV2.sol
├── MinerRegistry.sol
└── ProtocolCredits.sol

web/app/                     # Next.js frontend
├── page.tsx                 # Homepage
├── leaderboard/             # Agent rankings
├── jobs/                    # Job board
├── become-a-miner/          # → will become become-a-worker/
├── become-a-validator/      # → will become become-a-manager/
└── agent/[agentId]/         # Agent detail page
```

---

## Tech Stack

- **Python 3.10+** — core logic
- **FastAPI** — API server
- **Pydantic v2** — data contracts
- **Solidity 0.8.19** — contracts
- **web3.py** — Python → Base chain
- **Base Mainnet** — all contracts
- **pytest** — tests
- **Next.js 15 + React 19 + Tailwind** — frontend
- **Supabase** — API keys, registered agents, jobs, usage logs
- **LLM providers** (optional) — OpenAI, Anthropic, Ollama, Venice for intent verification

## Important Rules

- **The analyzer has no chain dependency** — pure Python. Don't add chain imports to analyzer.py, honeypot.py, or scorer.py.
- **System must work without chain** — API functions without RPC connection.
- **Spot checks are the moat** — no competitor has automated, objective quality measurement. Protect this.
- **"Job" not "task"** — everywhere, always. The protocol is job-agnostic.
- **Dynamic spot checks needed** — static templates can be memorized. Future work: LLM-generated spot checks.
- **x402 supports AVNC** — not just ETH/USDC.
- **Consensus scoring is dead code** — exists in scorer.py but forward.py never passes all_responses. Needs wiring up.
- **Don't push to main** — feature branches only.
- **No gh CLI** — push branch, give Jimmy the URL.

## Scoring (current vs planned)

**Current (live):** `0.6 * spot_check + 0.2 * consensus(DEAD) + 0.1 * format + 0.1 * speed`
**Planned:** `0.6 * spot_check + 0.25 * consensus(F1) + 0.10 * format + 0.05 * speed` + quality gate (>=0.70)

## Payment (current vs planned)

**Current (live):** Winner gets 85%, manager gets 15%. Losers get nothing.
**Planned (Gate+Base+Bonus):** Manager 15% off top. Remaining: 30% base pay pool (all who pass gate), 55% winner bonus, 15% reserve.

## Links

- Repo: https://github.com/JimmyNagles/agent-verification-network
- Domain: agentlabormarket (secured March 29, 2026)
- Contract: https://basescan.org/address/0x4e588353E7f247782A6109Fff3BA284a20D88c0F
- ERC-8004 TX: https://basescan.org/tx/0x38b165df227d6568f13e0d640a80220eaf35179ff03982b3740f2eda61c9b751

## Competitive Landscape

- **Closest competitor:** Moltlaunch (Base, ERC-8004, agent gig marketplace — but only client reviews, no automated QA)
- **Our moat:** Spot check mechanism — nobody else has it in the decentralized agent space
- **Key players:** Virtuals ACP (18K agents), Olas (10M+ txns), Fetch.ai (2M+ agents), Bittensor (ancestor)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JimmyNagles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
