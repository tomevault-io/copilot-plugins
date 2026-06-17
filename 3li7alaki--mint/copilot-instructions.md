## mint

> Structured development workflow that auto-routes coding tasks to the right execution mode — quick fixes, multi-step plans, research, shipping, or verification. Enforces quality gates (lint, types, tests), multi-stage code review, and atomic commits. Use when starting a feature, fixing a bug, refactoring code, planning implementation, shipping a release, or running quality checks.


# mint

Disciplined agentic development — fresh context per task, zero slop. Routes every coding task to the right execution mode, enforces quality gates, and manages multi-stage review pipelines.

## Quick Start

Invoke `mint` via the Skill tool with a task description. mint auto-detects the right mode:

| Mode | When | What it does |
|------|------|-------------|
| quick | Task touches ≤3 files, scope obvious | Direct implementation with quality gates |
| plan | Multi-file changes, ambiguous scope | Decomposes into specs, implements in phases |
| ship | Multiple features, "build all" | Parallel spec execution with coordinated review |
| research | "How to", "compare", investigation | Structured research with documented findings |
| verify | "Check gates", "audit" | Runs lint, types, tests and reports status |

## Routing Logic

The full orchestrator at `skills/mint/SKILL.md` evaluates signals in order — first match wins. Override with "just quick-fix it" or "plan this out" to force a mode.

## What mint Enforces

- **Quality gates** — lint + types + tests before every commit
- **Multi-stage review** — spec review, then parallel audit (quality, security, conventions, tests)
- **Atomic commits** — one commit per logical change, never push (human reviews and pushes)
- **Context protection** — heavy work delegated to subagents, main context stays clean

For the auto-trigger rule (invoke mint before any file modification), see `skills/using-mint/SKILL.md`.

---
> Source: [3li7alaki/mint](https://github.com/3li7alaki/mint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
