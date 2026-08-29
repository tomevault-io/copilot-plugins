## univer-sdk-skills

> This file provides context for AI agents working on this skill project.

# Univer Skills - Agent Guide

This file provides context for AI agents working on this skill project.

## Project Purpose

This repository contains AI agent skills for the [Univer](https://github.com/dream-num/univer) project. These skills help AI agents assist developers with:

1. Embedding Univer into existing web applications
2. Integrating Univer Pro enterprise features (collaboration, import/export, printing, pivot tables, charts, shapes)
3. Running Univer in Node.js backend environments for headless processing
4. Developing custom plugins for Univer

## Skill System Conventions

We follow the standard skill format:

```
skill-name/
├── SKILL.md           # Required. Frontmatter + instructions
├── agents/
│   └── openai.yaml    # UI metadata
├── references/        # Optional. Loaded on demand
├── assets/            # Optional. Templates, boilerplate
└── scripts/           # Optional. Executable utilities
```

### Writing Rules

1. **SKILL.md body < 500 lines** — split detailed content into references
2. **Frontmatter description is the trigger** — include all "when to use" info there
3. **References are loaded on demand** — always link them from SKILL.md with clear "when to read" guidance
4. **Code examples must be copy-paste ready** — assume the agent will emit them directly
5. **Side-effect imports must be explicit** — Univer's facade system requires `import '@univerjs/xxx/facade'`; never omit these

## When to Update Skills

- Univer releases a new major/minor version with breaking API changes
- A new official package is added (update `plugin-registry.md`)
- New facade methods are added (update `facade-api-guide.md`)
- Community reports a skill led to incorrect code (fix the reference)

## Testing Skills

After modifying skills, run validation:

```bash
# If your agent CLI provides a built-in validator (e.g. Codex):
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/univer-integrate
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/univer-plugin-dev
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/univer-pro-integrate
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/univer-node-backend

# Or use the standalone check script in this repo:
python3 scripts/validate.py skills/univer-integrate
python3 scripts/validate.py skills/univer-plugin-dev
python3 scripts/validate.py skills/univer-pro-integrate
python3 scripts/validate.py skills/univer-node-backend
```

## Forward-Testing

To verify a skill works on realistic tasks:

```bash
cd ~/univer-sdk-skills
# Test scaffold script
npx tsx skills/univer-plugin-dev/scripts/scaffold-plugin.ts test-plugin --path /tmp
```

## Maintenance Checklist

- [ ] SKILL.md frontmatter description is comprehensive and up-to-date
- [ ] All references linked from SKILL.md have correct relative paths (no cross-skill file paths)
- [ ] Side-effect import lists in `facade-api-guide.md` match latest packages
- [ ] Plugin registry lists all official `@univerjs/*` and `@univerjs-pro/*` packages
- [ ] Scaffold script generates valid, buildable code
- [ ] Templates use current package versions
- [ ] README.md and AGENTS.md reflect all existing skills

---
> Source: [dream-num/univer-sdk-skills](https://github.com/dream-num/univer-sdk-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
