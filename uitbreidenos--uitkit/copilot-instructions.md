## uitkit

> This file governs how Claude Code should behave when working inside this repository.

# CLAUDE.md — UitKit Repository Instructions

This file governs how Claude Code should behave when working inside this repository.

---

## What This Repo Is

UitKit is a knowledge system — a collection of skills, agents, hooks, rules, workflows, and prompts for Claude Code. The content here is *documentation and configuration*, not application code. When contributing or editing, treat correctness and clarity as the primary quality metrics.

---

## File Naming Conventions

- Skill files: `kebab-case.md` (e.g., `fastapi-crud.md`, `k8s-deployment.md`)
- Agent files: `kebab-case.md` (e.g., `security-reviewer.md`, `db-specialist.md`)
- Hook scripts: `kebab-case.sh` or `kebab-case.py`
- Guide files: `kebab-case.md`
- Workflow files: `kebab-case.md`
- Directories: `kebab-case/` (all lowercase, hyphenated)

---

## Skill File Format

Every file in `skills/` must follow this structure:

```markdown
# Skill Name

## When to activate
[Specific trigger conditions — be precise, not generic]

## When NOT to use
[Anti-patterns — when this skill is the wrong tool]

## Instructions
[The actual skill content — patterns, prompts, steps]

## Example
[At least one concrete example showing the skill in use]
```

Do not add sections beyond this structure without a clear reason.

---

## Agent File Format

Every file in `agents/` must follow this structure:

```markdown
# Agent Name

## Purpose
[One sentence — what domain or task this agent owns]

## Model guidance
[Haiku / Sonnet / Opus — and why]

## Tools
[Specific tool subset this agent should use — not all tools]

## When to delegate here
[Trigger conditions for spawning this agent]

## Example use case
[Concrete example]
```

---

## Hook File Format

Every file in `hooks/` must include:

1. A `.json` snippet showing the exact `settings.json` entry
2. The shell script (if applicable) with setup instructions
3. A description of what the hook does and when it fires

---

## Rules for Writing Content

- Write for a senior developer audience — no hand-holding, no filler
- Every claim about Claude Code behavior must be accurate as of the file's last-updated date
- No placeholder content — if a section isn't ready, omit it rather than writing "coming soon"
- Skills reference real Claude Code features (slash commands, hooks, agents, MCP) — not generic LLM advice
- Keep examples concrete and runnable, not abstract

---

## What Not to Do

- Do not create application code in this repo (no `src/`, no `app/`)
- Do not commit `development.md` — it is gitignored and private
- Do not add files outside the established directory structure without updating `README.md`
- Do not add emojis to file content unless it is a deliberate stylistic choice in a guide

---

## Language & Translation Rules

- **All content** must be translated into: French (`fr/`), German (`de/`), Dutch (`nl/`), Spanish (`es/`)
- This applies to: guides, skills, agents, rules, workflows, and prompts
- Translation files live in a language subdirectory inside the same parent: e.g., `skills/backend/python/fr/fastapi.md`, `guides/fr/getting-started.md`
- When updating an English file, note which translations need updating in the PR description
- Hook scripts (`.sh`/`.py`) are English-only — shell is universal

---

## Directory Ownership

| Directory | Purpose |
|---|---|
| `guides/` | Human-readable documentation — depth over brevity |
| `skills/` | Slash command definitions — one skill per file |
| `agents/` | Subagent YAML/MD definitions |
| `hooks/` | Event-triggered automations |
| `rules/` | Always-follow Markdown guidelines |
| `workflows/` | End-to-end multi-step process documentation |
| `prompts/` | Reusable prompt templates |
| `mcp/` | MCP server configs and recommendations |
| `examples/` | Complete, working project references |

---
> Source: [UitbreidenOS/UitKit](https://github.com/UitbreidenOS/UitKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
