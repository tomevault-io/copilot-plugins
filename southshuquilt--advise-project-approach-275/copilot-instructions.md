## advise-project-approach-275

> This repository packages one public skill: `advise-project-approach`.

# Repository Guidance

This repository packages one public skill: `advise-project-approach`.

## Source of Truth

- Edit the skill source at `skills/advise-project-approach/SKILL.md`.
- Do not edit `dist/advise-project-approach.skill` by hand. Rebuild it with `python scripts/package_skill.py`.
- Keep `.claude-plugin/plugin.json`, `CHANGELOG.md`, and release notes in sync when changing the public version.

## Validation

Before committing changes, run:

```bash
python scripts/validate_skill.py
python scripts/package_skill.py
python scripts/validate_skill.py
```

The validator checks:

- skill folder and required files exist
- frontmatter has only `name` and `description`
- skill name matches folder name
- description is within the trigger metadata budget
- packaged `.skill` archive contains the expected root folder and files

## Editing Rules

- Keep the packaged skill folder minimal: `SKILL.md` plus `agents/openai.yaml`.
- Keep human documentation at the repository root or in `examples/`, not inside the skill folder.
- Do not add runtime scripts to the skill package unless the skill truly needs executable behavior.
- Preserve the read-only, evidence-first safety posture of the skill.
- Avoid stale claims in examples. Do not hard-code star counts, release dates, or "latest" claims unless the example states they must be verified at review time.

---
> Source: [Southshuquilt/advise-project-approach-275](https://github.com/Southshuquilt/advise-project-approach-275) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
