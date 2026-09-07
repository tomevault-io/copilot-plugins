## gh-tracker

> Self-hosted GitHub analytics dashboard that captures every metric GitHub exposes.

# Project Intelligence — gh-tracker

## Overview
Self-hosted GitHub analytics dashboard that captures every metric GitHub exposes.
Stack: FastAPI (Python), SQLite, React + Tremor/Recharts frontend, Docker Compose.

**Critical**: GitHub Traffic API deletes data after 14 days. Automated archival is the #1 feature.

## Context Management

Hooks handle compaction automatically (PreCompact saves, PostCompact restores, SessionStart recovers).
You do NOT need to monitor context usage or manually compact.

**MEMORY.md discipline** — keep current at all times:
- `## Task Status:` line: `IN_PROGRESS`, `TASK_COMPLETE`, or `BLOCKED`
- Active task, completed items, next steps, file paths

**Subagent delegation** — keep main context lean:
- Exploratory searches and reads for understanding -> subagent (Agent tool)
- Reads for editing -> direct (Read tool)

---

## Autonomous Work Mode

Work autonomously. Don't narrate or ask permission — just do the work.

**Decision points**: Use AskUserQuestion with 2-4 concrete options, mark recommended with "(Recommended)".
**Don't ask**: "Should I proceed?" / "Is this okay?" — just do it or present specific options.

---

## Loop Breaker Protocol — MANDATORY

**The 3-Strike Rule**:
1. Max **3 attempts** at the same approach to solve the same problem.
2. After 3 strikes -> **mandatory pivot** to a different approach.
3. After pivot, max **2 more attempts**. If still stuck -> **ESCALATE**.

**Escalation chain**: retry (3x) -> pivot approach -> retry (2x) -> escalate to different agent -> ask user

---

## Team Agent Architecture

Seven-agent model. Agents: Architect, Builder, Tester, SecOps, DevOps, Verifier, Simplifier.

- Verification-first: Builder -> Verifier -> then review.
- SecOps has veto on security. Architect has veto on design/API.
- Builder defers to Tester on test adequacy.

---

## Project Standards

### Language: Python (Backend)
- Python 3.11+, FastAPI, httpx for async HTTP, SQLite via aiosqlite
- Testing: pytest + pytest-asyncio
- Linting: ruff
- Type hints required on all public functions
- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `ci:`

### Language: TypeScript (Frontend)
- React 18+, TypeScript strict mode
- Tremor for dashboard components, Recharts for charts
- Testing: vitest

### Architecture
- `backend/` — FastAPI app, collector, models, API routes
- `frontend/` — React dashboard
- `docker-compose.yml` — deployment
- SQLite DB in `data/` volume

### Security Baseline
- GitHub token via environment variable only, never in code
- HMAC-SHA256 webhook signature verification
- Input validation at all trust boundaries

---
> Source: [rayketcham-lab/gh-tracker](https://github.com/rayketcham-lab/gh-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
