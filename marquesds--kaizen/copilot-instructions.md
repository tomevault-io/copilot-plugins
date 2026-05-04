## agent-parity

> Agent parity — AGENTS.md as source of truth, .cursor/rules/ extensions, skill placement


# Agent Parity

Rules, skills, commands must be accessible to every agent: Cursor, Claude Code, Codex.

## Source of Truth

- `AGENTS.md` canonical config. `CLAUDE.md` points to it.
- `.cursor/rules/*.mdc` extend `AGENTS.md` with detailed rules.
- `.cursor/skills/<name>/SKILL.md` contain workflow guides.

## When Adding Rules or Skills

1. Add universal rules to `AGENTS.md` (or reference there)
2. Add Cursor-specific rules to `.cursor/rules/`
3. Add new skills to `.cursor/skills/<name>/SKILL.md`
4. Add skill to skills table in `AGENTS.md`
5. Never edit `CLAUDE.md` — it points to `AGENTS.md`

## When in Doubt

Rule only in `.cursor/rules/` but benefits other agents → add reference to `AGENTS.md`.

---
> Source: [marquesds/kaizen](https://github.com/marquesds/kaizen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
