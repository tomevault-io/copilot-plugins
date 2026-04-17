## interactive-tutorials

> This repo uses Cursor-format skills in `.cursor/skills/`. Each skill is a multi-phase workflow defined in a `SKILL.md` file. Available skills:

@AGENTS.md

## Cursor Skills

This repo uses Cursor-format skills in `.cursor/skills/`. Each skill is a multi-phase workflow defined in a `SKILL.md` file. Available skills:

- `.cursor/skills/autogen-guide/` — generate a guide from React/TypeScript source code
- `.cursor/skills/autogen-guide-dashboard/` — generate a guide from a Grafana dashboard JSON
- `.cursor/skills/migrate-guide/` — migrate a guide or learning path to the Pathfinder package format

When a user's request matches a skill trigger, read the skill's `SKILL.md` and follow its workflow. Skills write intermediate artifacts to `{output_dir}/assets/`; see `.cursor/skills/skill-memory.md` for the shared convention.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/grafana) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
