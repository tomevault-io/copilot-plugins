## claude-skills

> You are working in the `claude-skills` repository, which contains framework-agnostic AI agent skills used by both Gemini and Claude.

# GEMINI.md — claude-skills

## Primary Directive
You are working in the `claude-skills` repository, which contains framework-agnostic AI agent skills used by both Gemini and Claude.

**Read `CLAUDE.md` in this repository for all project rules, conventions, and skill documentation.**

## Gemini-Specific Guidelines

1. **Skill Installation:** Gemini natively loads skills from `~/.gemini/skills/`. Do **not** install or symlink skills into `~/.gemini/antigravity/skills/` as this will cause duplicate load warnings. Always use `./scripts/install-skills.sh` to manage symlinks across all agents automatically.
2. **Validation:** Always run `python3 scripts/audit_skills.py skills` before committing changes to ensure new or modified skills comply with repository standards (e.g., kebab-case naming, `SKILL.md` presence, no forbidden loose files).
3. **File Editing:** Edit files only in the `skills/` directory. Do not edit `.skill` ZIP files in `dist/`.
4. **Bug Filing:** If you encounter an `"UnhandledException"` from a skill script, read the `fix` field, document the bug in the skill's `KNOWN_BUGS.md`, and notify the user.
5. **Versioning:** If you modify a skill's logic or documentation, increment its semver version in the `SKILL.md` frontmatter.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Baphomet480) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
