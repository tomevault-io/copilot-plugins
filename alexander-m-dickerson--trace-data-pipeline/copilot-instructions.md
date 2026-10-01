## trace-data-pipeline

> - The skills above are slash commands: `/onboard`, `/run-wrds`, `/build-panel`,

@AGENTS.md

## In Claude Code

- The skills above are slash commands: `/onboard`, `/run-wrds`, `/build-panel`,
  `/reproduce-exhibits`, `/build-factors`, `/explain-pipeline`, `/add-a-column`. They live in
  `.claude/skills/`.
- `.claude/settings.json` lets the read-only checks run without asking (the tests, the dry
  runs, `doctor.py`, `tools/tags.py --check`) and blocks the usual spellings of a force-push
  and a recursive delete, in Bash and, on Windows, in PowerShell. It is a guard rail, not a
  sandbox, and a command run through the environment's Python by its path still asks.
  Claude Code applies it once you have accepted its "trust this folder" question.
- Each stage folder has its own `CLAUDE.md`, which Claude Code reads when it opens a file there.

---
> Source: [Alexander-M-Dickerson/trace-data-pipeline](https://github.com/Alexander-M-Dickerson/trace-data-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
