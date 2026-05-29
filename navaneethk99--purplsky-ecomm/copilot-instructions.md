## purplsky-ecomm

> This project uses the Payload CMS skill at `.agents/skills/payload/`.

# Agents

This project uses the Payload CMS skill at `.agents/skills/payload/`.
Start with `.agents/skills/payload/SKILL.md` for a quick reference, then see `.agents/skills/payload/reference/` for detailed docs.

When running commands, minimize output.

Use quiet flags and filters:
- Prefer `pytest -q` over `pytest -vv`
- Prefer `npm test -- --silent` over `npm test`
- Prefer `git diff --stat` before full `git diff`
- Prefer `rg`, `head`, `tail`, and `sed -n` over dumping whole files
- Pipe noisy output through `tail -120` or `grep -i error`

Never print huge logs, generated files, lockfiles, build folders, or dependency folders unless explicitly necessary.

---
> Source: [navaneethk99/purplsky-ecomm](https://github.com/navaneethk99/purplsky-ecomm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
