## research-planning-architect

> These rules apply to this skill folder.

# Skill Maintenance Rules

These rules apply to this skill folder.

- Keep `SKILL.md` concise and procedural; move long rubrics, examples, and reusable domain guidance into `references/`.
- Keep frontmatter limited to `name` and `description`; `name` must match the folder name.
- Make trigger descriptions specific enough to avoid generic research, writing, or coding tasks.
- Do not hard-code local absolute paths in scripts, examples, or templates.
- Scripts must be runnable with `--help`, avoid destructive defaults, and protect existing output files unless a force flag is used.
- After edits, run `python3 scripts/validate_skill.py .` and at least one representative script smoke test.
- Do not commit generated caches, temporary outputs, or test artifacts inside the skill folder.

---
> Source: [clear0215/research-planning-architect](https://github.com/clear0215/research-planning-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
