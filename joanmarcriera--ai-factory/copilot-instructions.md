## ai-factory

> This repo builds real projects using AI-assisted automation with minimal human intervention. The orchestrator (`orchestrator.py`) drives Aider to complete tasks defined in YAML files.

# AI Factory — Agent Rules

This repo builds real projects using AI-assisted automation with minimal human intervention. The orchestrator (`orchestrator.py`) drives Aider to complete tasks defined in YAML files.

## Core Principle
**Everything is codified.** Plans, conventions, expectations, and process rules live in workflows, skills, or this file — never in ad-hoc READMEs or chat context that gets lost.

## Repo Structure
```
orchestrator.py          # Task runner — reads tasks/*.yaml, calls Aider
tasks/                   # Task YAML files (XXX-description.yaml)
.agents/workflows/       # Process rules injected into prompts
  coding-conventions.md  # Per-project code style (injected into Aider)
  pre-task-guardrails.md # Automated post-execution checks
  project-bootstrap.md   # Checklist before creating task 001
  task-schema.md         # YAML schema rules for tasks
  use-uv.md              # Environment rules
.env                     # API keys (never committed)
src/                     # Generated project source code
config/                  # Project configuration files
tests/                   # Generated tests
```

## Rules for AI Agents

### 1. No freeform documentation
- Do NOT create README files, design docs, or notes unless explicitly requested.
- All process knowledge belongs in `.agents/workflows/` or `CLAUDE.md`.
- Project-specific READMEs are a task output, not an agent decision.

### 2. Workflows govern everything
- Before starting a new project: follow `.agents/workflows/project-bootstrap.md`.
- Before creating tasks: follow `.agents/workflows/task-schema.md`.
- Code conventions: defined per-project in `.agents/workflows/coding-conventions.md`.
- Guardrails: defined in `.agents/workflows/pre-task-guardrails.md`.

### 3. Task execution
- All work is done through task YAML files executed by the orchestrator.
- Never write project code directly — create a task and let the orchestrator run it.
- Exception: fixing orchestrator bugs or writing CLAUDE.md / workflow files.

### 4. Environment
- Use `uv run` for all Python execution (PEP 668 managed system).
- API keys are in `.env` with fallback chain in `AI_FALLBACK_MODELS`.
- Never hardcode API keys or model names in code.

### 5. Lessons learned
- After completing a project, update `.claude/projects/*/memory/lessons-learned-orchestrator.md`.
- After discovering a new failure pattern, add it to `pre-task-guardrails.md`.
- After discovering a validation anti-pattern, add it to `task-schema.md`.

### 6. Git discipline
- Each completed task is auto-committed by the orchestrator.
- Manual commits are for orchestrator changes, workflow updates, and CLAUDE.md edits.
- Never commit `.env` or API keys.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joanmarcriera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joanmarcriera)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
