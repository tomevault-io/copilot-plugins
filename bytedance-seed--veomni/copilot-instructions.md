## skills-reusable-only

> Only add reusable, maintainable skill definitions to .agents/skills/ following the Agent Skills standard


# Skills Directory: Agent Skills Standard

All skills under `.agents/skills/` must follow the [Agent Skills](https://agentskills.io) open standard:

- Each skill is a **folder** named `<skill-name>/` containing a `SKILL.md` file.
- `SKILL.md` must have YAML frontmatter with `name` (matching folder name) and `description`.
- Do NOT create flat `.md` files directly in `.agents/skills/` — only `README.md` is allowed.
- Optional subdirectories: `scripts/`, `references/`, `assets/`.
- Only add reusable, repeatable workflows. Put one-off scripts in `scripts/` or `tests/`.

---
> Source: [ByteDance-Seed/VeOmni](https://github.com/ByteDance-Seed/VeOmni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
