## spanish-translator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A multi-agent development pipeline using Claude Code's native agent/team system. Twelve specialized agents collaborate to build full-stack web (Next.js + Keystone backend) and mobile (Flutter) applications. Converted from a LangGraph StateGraph pipeline with full feature parity.

## Setup

```bash
# Option A: Setup script
/path/to/claude_teams/setup.sh /path/to/your-project

# Option B: Shell wrapper (add to ~/.zshrc)
source /path/to/claude_teams/sh-setup.md
```

## Running the Pipeline

```bash
cd /path/to/your-project

# Interactive agent with auto-resume (recommended)
claude_team

# Pipeline with task
claude_team "Build a todo app with authentication"

# Non-interactive / headless
claude_team --print "Build a REST API for user management"

# Resume after interruption
claude_team --resume

# Check pipeline progress
claude_team --status
```

Or use `.claude/run.sh` directly:
```bash
.claude/run.sh --interactive
.claude/run.sh "Build a todo app"
.claude/run.sh --print "Build a REST API"
.claude/run.sh --resume
.claude/run.sh --status
```

### run.sh Environment Overrides

| Variable | Default | Purpose |
|----------|---------|---------|
| `MAX_RESUMES` | 50 | Max auto-resumes before giving up |
| `COOLDOWN` | 15 | Base cooldown seconds between restarts |
| `MAX_COOLDOWN` | 7200 | Backoff cap (2 hours) |
| `MAX_TURNS` | 200 | Max turns per session (print mode) |
| `PRINT_MODE` | false | Headless mode |

## Architecture

### Pipeline Flow

```
User Request → Team Lead → Product Manager (Q&A → docs/PRD.md + docs/USER_JOURNEYS.md)
  → UI/UX Designer (Q&A → docs/DESIGN_SYSTEM.md + docs/USER_FLOWS.md; stays alive for dev consultations)
  → Gap Analysis (Team Lead cross-refs PRD + USER_JOURNEYS + USER_FLOWS → docs/GAP_ANALYSIS.md)
  → Architect (reads PRD + design docs → docs/ARCHITECTURE.md, declares branch + backend_needed)
  → [architecture_review checkpoint]
  → Branch Router
  → Keystone Backend Architect (if backend_needed: schemas, GraphQL, hooks, payments)
  → Web Dev / Mobile Dev (mandatory designer consult for new screens/flows) → Spec-Code Reconciliation
  → Validation (Ralph Wiggum loop, max 3 retries)
  → Code Simplifier (refine modified files for clarity + re-validate)
  → Code Reviewer (read-only, checks reconciliation) → QA Tester (Web/Mobile)
  → [pre_deploy checkpoint — user can skip deployment here]
  → DevOps Deployer (Q&A → docs/DEPLOYMENT.md; CI/CD, hosting, infrastructure)
  → docs/COMPLETE
```

For "web" with backend: backend runs first, then frontend (frontend consumes backend API).
For "both" branch: backend + mobile in parallel, then web frontend after backend, then shared code review and QA.
Deployment is optional — human can explicitly skip it at the `pre_deploy` checkpoint, and the project is marked complete without deployment artifacts.

### Agent Definitions

All agents live in `agents/` as markdown files with YAML frontmatter (`model`, `tools`). The **team-lead** is the orchestrator — it spawns teammates, relays messages, runs validation, and manages checkpoints. All other agents are spawned as teammates by the team lead.

| Agent | Role | Auto-loaded Skill |
|-------|------|-------------------|
| `team-lead` | Orchestrator + validation | — |
| `product-manager` | Requirements → PRD | — |
| `architect` | System design | — |
| `ui-ux-designer` | Design system, user flows, interaction maps | — |
| `keystone-backend-architect` | Backend schemas, GraphQL, hooks, payments | `keystone-backend-architect` |
| `web-developer` | Next.js frontend | — |
| `mobile-developer` | Flutter implementation | — |
| `code-simplifier` | Simplify code before review | — |
| `code-reviewer` | Code review + static analysis | — |
| `qa-tester-web` | Cypress E2E/component testing | `cypress-qa-engineer` |
| `qa-tester-mobile` | Flutter unit/widget/integration testing | `mobile-app-master` |
| `devops-deployer` | Deployment, CI/CD, infrastructure, app stores | — |

### State & Resume System

All state is persisted to `docs/` files — there is no in-memory state that matters:

| File | Purpose |
|------|---------|
| `docs/checkpoint.json` | Pipeline state (phase, branch, verdicts, retry counts). Updated every transition. Primary resume mechanism. |
| `docs/audit_log.md` | Master activity timeline |
| `docs/memory.json` | Cross-agent context (task, decisions, artifacts) |
| `docs/logs/<agent>.md` | Per-agent step logs for replacement agent handoff |
| `docs/PRD.md` | Product requirements (by PM) |
| `docs/ARCHITECTURE.md` | System design (by Architect) |
| `docs/DESIGN_SYSTEM.md` | Design tokens, colors, typography, spacing, animations (by Designer) |
| `docs/USER_FLOWS.md` | User flows with interaction maps for every screen/element (by Designer) |
| `docs/USER_JOURNEYS.md` | Structured user journeys with source attribution (by PM) |
| `docs/inferences.json` | Agent inference log with confidence signals (by all dev agents) |
| `docs/GAP_ANALYSIS.md` | Cross-reference gap analysis (by Team Lead, Phase 2.5) |
| `docs/discussions.md` | Inter-agent challenge discussions (append-only log) |
| `docs/DEPLOYMENT.md` | Deployment guide, CI/CD config, verification checklist (by DevOps Deployer) |
| `docs/deploy-state.json` | Granular deployment step progress — enables stop/resume at any point |
| `docs/COMPLETE` | Completion marker — stops the run.sh loop |

`.claude/run.sh` wraps `claude` with auto-resume: on exit it checks `docs/COMPLETE`, and if not found, restarts with `claude --continue` after exponential backoff. Crash loop detection stops after 3 fast failures (<30s each). Includes lock file (`docs/.pipeline.lock`) to prevent concurrent runs and pre-flight checks.

### Retry Loops

| Loop | Trigger | Max |
|------|---------|-----|
| Validation (Ralph Wiggum) | Lint/type errors | 3 retries |
| Code Review | NEEDS_CHANGES verdict | Back to dev |
| QA | FAILED verdict | Back to dev |
| Total iterations | Any dev retry | 10 max |

### Verdict Parsing

Code reviewer and QA tester end responses with a verdict keyword (APPROVED/NEEDS_CHANGES/REJECTED or PASSED/FAILED). Team lead uses **last-occurrence-wins** — find the last position of each keyword, highest position wins.

### Human Checkpoints

Configured via `human_checkpoints` field in `docs/checkpoint.json` (JSON array). Default enabled: `architecture_review`, `pre_deploy`. Optional: `pre_code`, `post_code`, `code_review`, `qa_review`, `ralph_wiggum_escalation`. At the `pre_deploy` checkpoint, the user can skip deployment entirely — the project is then marked complete without deployment artifacts.

## Modifying Agents

Edit the markdown files in `agents/`. YAML frontmatter controls model (`sonnet`/`opus`/`haiku`) and available tools. The body is the agent's system prompt — it defines behavior, responsibilities, and protocols.

Key pattern across all agents: **incremental checkpointing**. Every agent writes progress to disk immediately (draft files, step logs) so replacement agents can continue after token exhaustion.

---
> Source: [Junhanliu-dev/spanish-translator](https://github.com/Junhanliu-dev/spanish-translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
