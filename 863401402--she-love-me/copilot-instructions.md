## she-love-me

> - This repository packages the `she-love-me` workflow for analyzing WeChat or QQ chat histories and generating an HTML relationship report.

# AGENTS.md

## Repository Focus

- This repository packages the `she-love-me` workflow for analyzing WeChat or QQ chat histories and generating an HTML relationship report.
- The unified skill entrypoint for all tools is `.agents/skills/she-love-me/SKILL.md` (Claude Code, OpenClaw, Codex, Cursor, Copilot, Gemini CLI).
- Analysis knowledge is split across `references/` under that directory; SKILL.md is a control-plane only (~200 lines).

## Codex Guidance

- When a user asks to analyze chat logs with this project, prefer the repo skill `she-love-me`.
- Keep the working directory at the repository root when following the skill workflow.
- Keep generated or sensitive outputs under `vendor/`, `data/`, and `reports/`; do not move personal chat data into tracked files.
- If the user wants to invoke the skill explicitly in Codex, they can mention `$she-love-me`.

---
> Source: [863401402/she-love-me](https://github.com/863401402/she-love-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
