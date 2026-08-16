## planit

> **`.specify/memory/constitution.md` governs all work in this repository.** Read it before

# CLAUDE.md

## Source of truth

**`.specify/memory/constitution.md` governs all work in this repository.** Read it before
specifying, planning, or writing code — it defines the six core principles, the product and
engineering constraints, and the review gates a change must pass to be mergeable.

This file deliberately does not restate those rules. It records only where things are and how to
operate the repo. If anything here, in a prompt, or in a tool default conflicts with the
constitution, the constitution wins.

Amend the constitution only through `/speckit-constitution` — never by editing principles inline
as part of feature work.

## Repository layout

- `.specify/` — Spec Kit workspace: `memory/constitution.md` (source of truth), `templates/`,
  `scripts/`, `workflows/`
- `static/` — the branding kit every screen must apply: `planit.css`, `planit-logo.svg`,
  `planit-logo-dark.svg`, `planit-globe.svg`, `PlanIt-Hero.png`, `flags/*.svg`
  (also archived at `planit-branding-kit.zip`)
- `.claude/skills/` — the Spec Kit slash commands available in this session
- `.venv/` — project virtual environment

## Current state

Greenfield. `main` has no commits, and there is no application code, no test suite, and no
`pyproject.toml` or `requirements.txt`. `.venv` exists but contains only `pip` — FastHTML and
pytest are not installed yet, so the first feature's plan needs to declare them.

## Working in this repo

Spec-driven flow, in order:

```
/speckit-specify → /speckit-clarify → /speckit-plan → /speckit-tasks → /speckit-implement
```

`/speckit-analyze` (cross-artifact consistency), `/speckit-checklist`, and `/speckit-converge`
are also available. Feature numbering is sequential; helper scripts are Python (`.specify/scripts/`).

## Commands

```bash
source .venv/bin/activate    # always work inside the venv
python -m pytest             # test suite (once it exists)
```

---
> Source: [dannycastro919/planit](https://github.com/dannycastro919/planit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
