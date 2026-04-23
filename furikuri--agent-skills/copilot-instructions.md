## agent-skills

> This repository is a collection of reusable agent skills for Claude Code.

# Agent Skills Collection

This repository is a collection of reusable agent skills for Claude Code.

## Repo Structure

```
skills/
  <skill-name>/
    SKILL.md          # Skill definition (frontmatter + instructions)
```

Each skill resides in its own directory under `skills/`. The `SKILL.md` file contains YAML frontmatter (`name`, `description`) followed by the skill instructions.

## Conventions

- **Language**: Skills are written in English
- **SKILL.md Frontmatter**: Each skill requires `name` and `description` in the YAML frontmatter
- **No external dependencies**: Skills should work standalone and only use tools natively provided by Claude Code
- **Directory name = Skill name**: The folder name under `skills/` corresponds to the skill identifier

## Existing Skills

- `java-repo-assessment` — Comprehensive quality analysis of Java repositories (static analysis, architecture metrics, Git forensics)
- `claude-ipc` — Communicate with other Claude Code sessions via tmux (send prompts, wait for completion, capture results)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FuriKuri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
