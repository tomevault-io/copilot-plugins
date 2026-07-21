## spec-driven-develop

> These instructions apply to the whole repository.

# Project Agent Instructions

## Scope

These instructions apply to the whole repository.

## Truth Sources

- Core workflow: `plugins/spec-driven-develop/skills/spec-driven-develop/SKILL.md`
- Shared workflow references: `plugins/spec-driven-develop/skills/spec-driven-develop/references/`
- Claude Code sub-agent prompts: `plugins/spec-driven-develop/agents/`
- User-facing docs: `README.md` and `README.zh-CN.md`
- Persistent project memory: use the active coding agent's native project memory surface when available. Do not add a repository fallback memory file unless the workflow explicitly records that choice.

## Development Rules

- Keep this repository Markdown-first. Do not add runtime dependencies unless a requested feature truly needs executable tooling.
- For workflow behavior changes, update the core `SKILL.md`, the affected references/templates, and the affected agent prompts together. Avoid one-line prompt patches that leave other execution surfaces stale.
- When changing user-visible behavior, keep `README.md` and `README.zh-CN.md` consistent.
- `AGENTS.md` and `CLAUDE.md` are active project-level instruction surfaces. Keep them aligned; use `CLAUDE.md` only for Claude Code-specific notes and point back here for shared rules.
- New features or behavior changes must add or update relevant tests when an automated test surface exists. If no automated suite exists, run the closest static/syntax checks and document why no test was added.
- Record stable cross-session decisions in the native project memory surface when one is available. If none is available, record only agent-facing rules in `AGENTS.md`/`CLAUDE.md`; do not invent a repo-local memory file without an explicit workflow decision.

## Validation

Use the closest checks for the changed surface:

- `python -m py_compile scripts/export-progress.py`
- `bash -n scripts/install-codex.sh scripts/install-cursor.sh scripts/install-opencode.sh scripts/install-all.sh`
- Targeted `rg` checks for newly required workflow language
- `git diff --check`

---
> Source: [zhu1090093659/spec_driven_develop](https://github.com/zhu1090093659/spec_driven_develop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
