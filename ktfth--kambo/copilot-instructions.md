## kambo

> Kali Linux-based MCP server for evidence-driven penetration testing and bug bounty hunting.

# Kambo — Elite Bug Bounty MCP Server

## Project Overview

Kali Linux-based MCP server for evidence-driven penetration testing and bug bounty hunting.
All tools run inside a Docker container. Evidence chains grade findings as CONFIRMED/FIRM/TENTATIVE.

## Architecture

- `src/kambo/` — Core implementation (Python, async, Pydantic models)
- `src/kambo/tools/` — MCP tool implementations by phase
- `src/kambo/validation.py` — Evidence-based validators (8 vuln types)
- `src/kambo/bounty_pricing.py` — Finding value estimation
- `src/kambo/calibration.py` — Self-tuning from user feedback
- `src/kambo/pattern_analyzer.py` — Elite pattern detection
- `src/kambo/learnings.py` — Cross-session memory (JSONL)
- `tests/` — 200+ tests, all passing

## Conventions

- Python 3.11+ (CI tests 3.11–3.14), async/await everywhere
- Pydantic models with `frozen=True` for immutability
- Evidence chains: weighted signals accumulate to confidence levels
- Metrics tracked per-tool across sessions via SQLite
- No hardcoded paths — use config or environment variables
- Tests required for all new functionality

## Skill Routing

When the user's request matches an available skill, invoke it via the Skill tool.

| Pattern | Skill |
|---------|-------|
| Start hunting, bug bounty, autonomous hunt | `/kambo-hunt` |
| Refine, self-polish, improve kambo, auto-improve | `/kambo-refine` |
| Calibrate, adjust weights, recalibrate, tune | `/kambo-calibrate` |
| Report, generate report, bounty report, submit | `/kambo-report` |
| Crafter loop, auto improve loop, self-improve | `/kambo-loop` |
| Visualize, diagram, mermaid, show progress | `/kambo-viz` |
| Tutorial, teach, how to use, walkthrough | `/kambo-awake` |
| Code review, check quality, review kambo | `/kambo-review` |
| Confidence check, should I report, impact analysis, question findings | `/kambo-confidence` |
| One-liner, recipe, KingOfBugBounty, OFJAAAH, certstream, technique catalog | `/kambo-kingrecon` |
| Think like defense, defender perspective, blind spots, what they missed, lateral vectors, assumption gaps | `/kambo-think-like-defense` |
| JS analysis, client-side secrets, source maps, hidden endpoints, extract from JS | `/kambo-js-hunt` |
| Business logic, logic bugs, flow bypass, price manipulation, workflow abuse | `/kambo-logic-hunt` |
| WAF bypass, WAF blocking, evade, cloudflare bypass, payload blocked | `/kambo-waf-evade` |
| Chain exploits, multi-step, combine findings, escalate impact, attack chain | `/kambo-chain` |
| Race condition, TOCTOU, concurrent requests, double spend, timing attack | `/kambo-race` |
| GraphQL, introspection, query batching, nested queries, graphql security | `/kambo-graphql` |
| Post social, publicar Threads, post do dia, divulgar kambo | `/kambo-social` |

## Self-Improvement Loop

The system learns from every session:
1. **Metrics** track per-tool precision and FP rates (SQLite)
2. **Pattern analyzer** classifies tools as elite/reliable/noisy/broken
3. **Calibration engine** detects prediction-reality drift
4. **Learnings store** persists insights across sessions (JSONL)
5. **Skills** orchestrate the improvement cycle

After each hunting session: run `/kambo-refine` to analyze, then `/kambo-calibrate` to tune.

## Crafter Loop

For continuous autonomous improvement, use `/kambo-loop` inside Ralph Loop:
```
/ralph-loop "/kambo-loop" --max-iterations 30 --completion-promise DONE
```

Each iteration: assess → plan → implement → verify → persist. One change per cycle.
Learnings store at `~/.kambo/learnings.jsonl` provides continuity between iterations.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues for `ktfth/kambo` (via the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Canonical triage roles map 1:1 to label strings (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout (`CONTEXT.md` + `docs/adr/` at the repo root). See `docs/agents/domain.md`.

---
> Source: [ktfth/kambo](https://github.com/ktfth/kambo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
