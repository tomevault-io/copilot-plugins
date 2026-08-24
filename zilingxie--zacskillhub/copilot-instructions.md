## zacskillhub

> This repository is for creating and maintaining reusable Codex skills. Keep the tree organized around skill folders instead of general project docs or application code.

# AGENTS.md

## Scope

This repository is for creating and maintaining reusable Codex skills. Keep the tree organized around skill folders instead of general project docs or application code.

## Repository Structure

- Put each skill in its own top-level directory named in kebab-case.
- Required file: `<skill-name>/SKILL.md`
- Optional subdirectories:
  - `<skill-name>/agents/openai.yaml`
  - `<skill-name>/scripts/`
  - `<skill-name>/references/`
  - `<skill-name>/assets/`

## SKILL.md Rules

- Use YAML frontmatter with only `name` and `description`.
- Keep `name` in lowercase kebab-case.
- Start `description` with `Use when...`.
- Write `description` as trigger guidance, not a workflow summary.
- Keep `SKILL.md` concise and move heavy reference material into `references/` when needed.

## Authoring Guidance

- Prefer reusable scripts for deterministic or repetitive tasks.
- Keep bundled references one level away from `SKILL.md` and link them directly from the main file.
- Refresh `agents/openai.yaml` when a skill changes and the UI metadata is no longer accurate.
- Do not add auxiliary docs such as `README.md`, `CHANGELOG.md`, or setup guides unless explicitly requested.

## Working Rules

- Inspect the current repo state before adding or editing a skill.
- If the repository is still empty, bootstrap only the minimum structure needed for the requested task.
- Use `rg` for search and `apply_patch` for manual file edits.
- Keep commits focused and small.
- Never revert unrelated user changes.
- Verify filenames, relative paths, and internal links before committing.

## Git

- Default branch: `main`
- Prefer concise commit messages such as:
  - `docs: add repo agent instructions`
  - `feat: add <skill-name> skill`
  - `docs: refresh <skill-name> metadata`

---
> Source: [ZilingXie/ZacSkillHub](https://github.com/ZilingXie/ZacSkillHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
