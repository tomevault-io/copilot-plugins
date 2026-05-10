## blitz-bounty-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Autonomous smart contract security toolkit powered by Claude Code orchestration. Two modes:
- **Audit competitions** (Cantina, Sherlock, Code4rena, CodeHawks) — all code is target, duplicates pay, Foundry mock PoC
- **Immunefi bug bounties** — post-audit code focus, dup check before PoC, mainnet fork only

All flows use `claude -p` subprocess orchestration (subscription-based, no API key required).

## Commands

### Primary (use these)

```bash
# Single-run structured hunt (auto-detects audit vs bounty from URL)
/web3-hunt "https://cantina.xyz/competitions/..." cantina
/web3-hunt "https://immunefi.com/bug-bounty/balancer" immunefi "https://1rpc.io/eth"

# Autonomous loop with coverage gap tracking
/web3-loop "https://cantina.xyz/competitions/..." cantina 10
/web3-loop "https://immunefi.com/bug-bounty/balancer" immunefi 10 "https://1rpc.io/eth"
```

### Headless (background/CI, no interactive Claude session)

```bash
python3 hunt_loop.py <immunefi-url> <rpc-url> [max_iterations]
python3 audit_loop.py <contest-url> <platform> [max_iterations] [rpc-url]
python3 audit_orchestrator.py <source-dir> [--platform codearena] [--timeout 900]
python3 audit_orchestrator.py <source-dir> --architecture cache-multipass  # sequential lens-based passes
```

### Benchmarks

```bash
# EVMBench detect mode — multiple strategies
python3 benchmark/evmbench_skill_runner.py --split detect-tasks                    # skill (default)
python3 benchmark/evmbench_skill_runner.py --audit 2024-04-noya --strategy raw     # raw baseline
python3 benchmark/evmbench_skill_runner.py --audit 2024-04-noya --strategy checklist  # structured checklist
python3 benchmark/evmbench_skill_runner.py --audit 2024-04-noya --strategy tiered  # Sonnet sweep → Opus deep
python3 benchmark/evmbench_skill_runner.py --strategy tiered --sweep-model haiku   # Haiku sweep variant
python3 benchmark/evmbench_skill_runner.py --compare

# EVMBench patch mode (find + fix)
python3 benchmark/evmbench_patch_runner.py --post-cutoff
python3 benchmark/evmbench_patch_runner.py --audit 2026-01-tempo-feeamm

# Custom suites
python3 benchmark/run.py --suite known-vulns

# EVMBench official harness (requires Docker + uv)
bash benchmark/evmbench_setup.sh  # one-time setup
```

### Foundry PoC

```bash
cd poc-forge && forge test --match-contract Finding_PoC -vv
```

## Architecture

```
User/CI ──┬── Interactive (/web3-hunt, /web3-loop) ── Claude Code slash commands
          └── Headless (hunt_loop.py, audit_loop.py) ── subprocess calling `claude -p`

Both feed into the same phase pipeline:
  Recon → Systematic File Sweep (100% coverage) → Cross-Compare → Attack Surface → Triage → PoC → Report
```

**Headless orchestrators** (`hunt_loop.py`, `audit_loop.py`, `audit_orchestrator.py`) invoke `claude -p` as subprocesses. They parse structured signals from Claude's output (e.g., `HUNT_SIGNAL:FOUND:`, `AUDIT_SIGNAL:DROP:`) to drive loop state. State persists in `hunt-state.json` / `audit-state.json`.

**`audit_orchestrator.py`** is a multi-agent variant: runs parallel specialist agents (recon → parallel vulnerability analysis → merge/triage) via `ThreadPoolExecutor`.

**Benchmark system:**
- `evmbench_skill_runner.py` — detect mode, wraps EVMBench through `/web3-hunt` skill
- `evmbench_patch_runner.py` — patch mode, agent finds + fixes vulns, graded by forge tests
- `evmbench_common.py` — shared audit inventory, config, cloning utilities
- `llm_judge.py` — Haiku-based semantic matcher for scoring against ground truth
- `run.py` — custom benchmark suites defined in `benchmark/suites/`

## Key Rules for Vulnerability Hunting

- **100% file coverage.** Read EVERY in-scope .sol file. Peripheral files hide the best bugs.
- **Cross-compare pattern groups.** If 20 connectors implement the same interface, compare all 20. The bug is the one that differs.
- **Attack scenario first.** Never start from "this code looks weird." Start from "what can an attacker DO?"
- **Asymmetry != bug.** Find the technical reason before calling it a bug.
- **Trace every value function.** TVL, balance, price, shares — trace the math.
- **Economic feasibility.** Include gas vs profit analysis with optimal parameters.
- **Bounty mode:** dup check BEFORE PoC (biggest time waster). PoC = mainnet fork + web3.py verification.
- **Audit mode:** speed matters (contest windows close). High AND Medium both count.

## RPC

- Ethereum: `https://1rpc.io/eth`
- No mainnet testing — fork only (Immunefi rule)

## Prerequisites

- Claude Code CLI, Foundry (`forge`/`cast`/`anvil`), Python 3.10+, PyYAML
- Optional: Docker + uv (EVMBench official harness), `gh` CLI (Sherlock auto-submit)

## Auth

All flows use `claude -p` on a logged-in Claude Code subscription session. No `ANTHROPIC_API_KEY` required.
Quick check: `python3 benchmark/claude_subscription_check.py --probe`

---
> Source: [sane100400/Blitz-Bounty-Agent](https://github.com/sane100400/Blitz-Bounty-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
