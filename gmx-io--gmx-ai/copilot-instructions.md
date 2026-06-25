## gmx-ai

> Skill files are duplicated across three directories for compatibility with different indexing systems:

# Agent Instructions

## Skill File Synchronization

Skill files are duplicated across three directories for compatibility with different indexing systems:

| Directory | Purpose |
|-----------|---------|
| `.well-known/skills/` | Web-based skill discovery (well-known URI) |
| `skills/` | Direct skill installation (`npx skills add`) |
| `plugins/gmx-io/skills/` | Claude Code plugin marketplace |

**When modifying any skill file, apply the same change to all three locations.**

### Current skills

- `gmx-trading/` — SKILL.md + references/
- `gmx-liquidity/` — SKILL.md + references/

---
> Source: [gmx-io/gmx-ai](https://github.com/gmx-io/gmx-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
