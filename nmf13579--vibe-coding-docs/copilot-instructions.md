## vibe-coding-docs

> Docs-first core — read order, routing, and post-change updates for Vibe-coding-docs


# Core (Vibe-coding-docs)

This workspace is a docs-first operating system for vibe coding.

## Session lifecycle

onSessionStart (every new session — automatic, no command needed):
  - Read HANDOFF.md and LAYER-3/project-status.md
  - Read LAYER-3/lessons.md if it exists
  - If same task unclosed again → warn: "⚠️ Задача [name] идёт уже несколько сессий. Разбить или продолжить?"
  - Summarize current state in 3-5 lines
  - Suggest next 1-3 actions
  - Wait for owner's choice — do not write code until confirmed

onTaskComplete (after every completed task — automatic):
  - Update HANDOFF.md and LAYER-3/project-status.md
  - Update additional files per CLAUDE.md Автосохранение table
  - Confirm: "Контекст сохранён ✅"

onSaveReminder (remind if any of these):
  - 10+ file edits since last HANDOFF.md update
  - Plan of 3+ steps completed without updating HANDOFF.md
  → remind: "⚠️ Контекст не сохранялся. Сохранить сейчас?"

onAuditRequest ("проверь проект", "аудит", "здоровье"):
  - Read and run LAYER-1/audit.md protocol

onFeatureRequest ("что добавить", "что дальше", after phase close):
  - Read LAYER-1/feature-radar.md + LAYER-3/features.md
  - Suggest top 3 with reasoning

## Read first

1. `README.md`
2. `llms.txt / LAYER-1/workflow.md`
3. `CLAUDE.md`
4. `HANDOFF.md`

## Route by task

- Project startup (new project or existing code) → `llms.txt / LAYER-1/workflow.md` Этап 0, `LAYER-1/agent-bootstrap.md`
- Communication style → `LAYER-1/dialog-style.md`, `LAYER-1/glossary.md`
- Planning → `LAYER-2/specs/planning.md`, `LAYER-2/specs/roadmap.md`
- Architecture → `LAYER-2/specs/architecture.md`, `LAYER-1/stack-presets.md`
- Review → `LAYER-1/task-protocol.md`
- Audit → `LAYER-1/audit.md`
- Security → `LAYER-1/security.md`, `LAYER-3/security.md`
- Product suggestions → `LAYER-1/feature-radar.md`, `LAYER-3/features.md`
- Recovery after confusion → `LAYER-1/context-recovery.md`

## Notes

Specialized sub-agents (audit-agent, feature-advisor-agent, session-guard-agent)
are available only in Claude Code and cannot be invoked from Cursor.
Use the corresponding docs directly:
audit → `LAYER-1/audit.md`, features → `LAYER-1/feature-radar.md`,
context guard → follow `CLAUDE.md` save triggers manually.

## Working principles

- Prefer existing repository docs over assumptions.
- Keep suggestions practical and prioritized.
- Avoid introducing unnecessary complexity.

## Update after major changes

- `HANDOFF.md`
- `LAYER-3/project-status.md`
- `LAYER-3/deferred-decisions.md` if a decision is postponed
- `LAYER-3/fixes.md` if a reusable fix was found

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NMF13579) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
