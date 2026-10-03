## project-handoff-skill

> - Keep distributable Codex skills under `skills/<skill-name>/`.

# Repository rules

- Keep distributable Codex skills under `skills/<skill-name>/`.
- Treat `skills/project-handoff/` as the canonical published copy of the skill.
- Keep repository-level documentation at the repository root; do not place README or changelog files inside the skill directory.
- Do not commit credentials, machine-specific paths, generated caches, handoff packets, or test artifacts.
- Validate the skill with `quick_validate.py` and run the bundled validator tests before publishing changes.
- Keep changes narrowly scoped and preserve backward compatibility for existing handoff packets unless a breaking change is explicitly approved.

---
> Source: [somnus-J-307/project-handoff-skill](https://github.com/somnus-J-307/project-handoff-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
