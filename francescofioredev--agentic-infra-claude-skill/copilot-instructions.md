## agentic-infra-claude-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

This repository contains the **`agentic-infra` Claude Code Skill** — a knowledge base for designing, reviewing, and linting agentic AI infrastructure. The skill is grounded entirely in `docs/Agentic_Design_Patterns.pdf` (482 pages, 21 chapters + appendices).

## Repository Structure

```
.
├── PLAN.md                          # PDF index, structure, self-review status
├── docs/
│   └── Agentic_Design_Patterns.pdf  # Source of truth (sole evidence source)
├── agentic-infra.skill              # Packaged distributable skill file
└── .claude/skills/agentic-infra/    # Skill source directory
    ├── SKILL.md                     # Skill metadata + usage instructions
    ├── taxonomy.yaml                # concern → patterns mapping
    ├── patterns/                    # 21 fully-detailed pattern cards (01-21)
    ├── checklists/review.md         # 62-item architecture review rubric
    ├── templates/                   # ADR template + review report template
    ├── scripts/lint_agentic_arch.py # Deterministic ADR linter (17 rules)
    └── examples/                    # Design example + review example
```

## Key Commands

**Run the lint script:**
```bash
python3 .claude/skills/agentic-infra/scripts/lint_agentic_arch.py <path/to/adr.md>
python3 .claude/skills/agentic-infra/scripts/lint_agentic_arch.py <path/to/adr.md> --format text
```
Exit code: 1 if any CRITICAL finding, 0 otherwise.

**Package the skill** (requires `pyyaml`; use a venv):
```bash
python3 -m venv /tmp/skill_venv
/tmp/skill_venv/bin/pip install pyyaml -q
/tmp/skill_venv/bin/python3 /path/to/skill-creator/scripts/package_skill.py \
  .claude/skills/agentic-infra/ .
```

## Evidence Policy

Every rule, recommendation, or best practice in this skill cites a page number from `docs/Agentic_Design_Patterns.pdf`. Format: `(p. N)`. No external sources are used.

## Permissions

`.claude/settings.local.json` pre-approves `Bash(brew install:*)` for installing dependencies via Homebrew.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/francescofioredev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/francescofioredev)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
