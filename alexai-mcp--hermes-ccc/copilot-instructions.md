## hermes-ccc

> This package is a Claude Code native skill pack. Treat every file in this directory as distributable content, not scratch space.

# hermes-CCC Maintainer Notes

This package is a Claude Code native skill pack. Treat every file in this directory as distributable content, not scratch space.

## Goals

- Port high-value Hermes Agent operating patterns into Claude Code skills.
- Keep the package installable with no extra build step.
- Make every skill immediately actionable for engineering, research, or tool-use workflows.
- Prefer practical commands, decision rules, output contracts, and failure handling over marketing prose.

## Package Rules

- Every skill must live in `skills/<skill-name>/SKILL.md`.
- Every `SKILL.md` must contain YAML frontmatter with:
  - `name`
  - `description`
  - `version`
  - `author`
  - `license`
  - `metadata`
- Every skill body must remain at or above `100` lines of substantive instructions.
- Every skill description must say what the skill does and when to use it.
- Every skill should include:
  - purpose
  - activation criteria
  - procedure
  - decision rules
  - outputs
  - failure modes
  - concrete commands or templates when applicable

## Content Style

- Write in imperative voice.
- Use flat bullets and numbered lists only where they clarify execution order.
- Avoid vague advice such as "be careful" without explaining how to operationalize it.
- Prefer exact commands, file paths, API endpoints, JSON shapes, or prompt patterns.
- Keep examples realistic and tool-specific.

## Scope Boundaries

- Do not add package-level dependencies unless the package actually needs them.
- Do not assume network access in the installer scripts.
- Do not store secrets, tokens, or local machine paths that only work on one workstation.
- Do not reference internal tooling that a typical Claude Code user would not have.

## Validation Expectations

Before shipping changes, verify:

1. required root files exist
2. all skill folders exist (currently 46; CI checks dynamically)
3. every `SKILL.md` parses as YAML frontmatter plus Markdown body
4. every skill contains the required frontmatter keys
5. every skill body is long enough and still coherent
6. `install.sh` and `install.ps1` both parse cleanly

The GitHub Actions workflow should remain the source of truth for structural validation.

## Editing Guidance

- Preserve stable skill names; they are install paths.
- Expand skills by adding procedures and commands, not filler.
- If you change a skill's operational contract, update `README.md`, `docs/tool-mapping.md`, and `docs/migration-guide.md` when needed.
- If you add or remove skills, update the full skills table in `README.md`. CI validates skill count dynamically (no hardcoded number).

---
> Source: [AlexAI-MCP/hermes-CCC](https://github.com/AlexAI-MCP/hermes-CCC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
