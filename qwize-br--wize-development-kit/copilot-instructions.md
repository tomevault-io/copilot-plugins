## wize-create-skill

> workflow: Create Skill


# Create Skill

# Create Skill

**Goal.** Scaffold a single-step skill in `.wize/custom/skills/{code}/`.

## Inputs (interactive)
- `code`, `name`, `description`
- `owner` (which agent owns this skill)
- `inputs[]`, `outputs[]`
- `body` (the actual skill content / prompt template)

## Outputs
- `.wize/custom/skills/{code}/skill.md`
- IDE adapter regeneration.

## Validation
- Schema (`schemas/skill.schema.json`).
- Markdown lint.
- Dry-run with stub input.

---
> Source: [qwize-br/wize-development-kit](https://github.com/qwize-br/wize-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
