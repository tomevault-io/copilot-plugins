## skills-manager

> - Keep `SKILL.md` concise and treat `references/user-guide.md` as the detailed user-facing guide.

# Skills Manager development

- Keep `SKILL.md` concise and treat `references/user-guide.md` as the detailed user-facing guide.
- Preserve the `scripts/skills_manager.py` command interface unless a change is explicitly requested; filesystem mutations remain dry-run by default and require `--apply`.
- Use only the Python standard library in the manager script and preserve atomic writes, recoverable backups, and rollback behavior.
- After changes, run `python3 scripts/test_skills_manager.py` and the bundled `skill-creator` `quick_validate.py` against this directory.
- Do not add unrelated documentation, dependencies, Git setup, or plugin-management behavior.

---
> Source: [zuco1111/skills-manager](https://github.com/zuco1111/skills-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
