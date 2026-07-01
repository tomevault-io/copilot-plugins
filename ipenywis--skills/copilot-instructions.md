## skills

> This repository contains portable `SKILL.md` skills for coding agents.

# Repository Instructions

This repository contains portable `SKILL.md` skills for coding agents.

## Skill Authoring

- Keep every skill concise, portable, and useful without private context.
- Use only `name` and `description` in `SKILL.md` frontmatter.
- Put triggering guidance in `description`; the body is loaded only after a
  skill triggers.
- Add `scripts/`, `references/`, or `assets/` only when the skill needs them.
- Do not claim these skills are official guidance from any model provider or
  public speaker.

## Safety Boundary

Skills that guide coding-agent work must require explicit written approval
before edits, installs, migrations, commits, deploys, deletes, or other
mutating work.

## Validation

Validate changed skills with the Codex skill validator when available:

```bash
python3 /Users/islemmaboud/.codex/skills/.system/skill-creator/scripts/quick_validate.py <path-to-skill>
```

---
> Source: [ipenywis/skills](https://github.com/ipenywis/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
