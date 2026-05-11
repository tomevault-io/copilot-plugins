## semibold-skills

> This repository contains **Claude Code skills** published by [Semibold](https://semibold.cz). When editing or adding skills, follow these rules.

# Guidance for AI Agents Working in This Repo

This repository contains **Claude Code skills** published by [Semibold](https://semibold.cz). When editing or adding skills, follow these rules.

## Repo structure

- **skills/** — Each subdirectory is one skill. Agents discover skills by scanning `skills/` for directories that contain `SKILL.md`.
- **Skill directory name** must exactly match the `name` in that skill's frontmatter (e.g. `skills/audit-links/` ↔ `name: audit-links`).

## SKILL.md requirements

- **Frontmatter (YAML):**
  - `name` (required): lowercase, hyphens only, max 64 chars, must match parent directory name.
  - `description` (required): what the skill does and when to use it; include trigger terms so agents know when to apply it. Max 1024 chars.
- **Body:** Markdown instructions in English. Keep under ~500 lines; put long reference material in `references/` or supporting code in `scripts/` and link from SKILL.md.

## Conventions

- Write descriptions in third person ("Use when…" not "You can use when…").
- Be concise; focus on correct usage, pitfalls, and required confirmations.
- When adding a new skill: create `skills/<skill-name>/SKILL.md`, then update `README.md` (skills table) and `skills/llms.txt` (index + triggers).
- Never commit secrets, API keys, or client-specific data.

## References

- [Agent Skills specification](https://agentskills.io/specification.md)
- [Claude Code skills docs](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/skills)
- [skills CLI (discovery, install)](https://github.com/vercel-labs/skills)

---
> Source: [semibold-cz/semibold-skills](https://github.com/semibold-cz/semibold-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
