## agentic-stack-desktop

> Autohand Code CLI reads `AGENTS.md` as project instructions. This file

# AGENTS.md — Autohand Code CLI adapter for agentic-stack

Autohand Code CLI reads `AGENTS.md` as project instructions. This file
points it at the portable `.agent/` brain so Autohand shares the same
memory, skills, and protocols as the other installed harnesses.

Before acting:

1. Read `.agent/AGENTS.md`.
2. Read `.agent/memory/personal/PREFERENCES.md` when it exists.
3. Use `.agent/tools/recall.py "<task intent>"` for relevant accepted
   lessons on non-trivial work.
4. Prefer skills from `.agent/skills/` when their triggers match.

After meaningful work, record a short reflection through
`.agent/tools/memory_reflect.py` so the normal dream/review cycle can
stage candidate lessons.

---
> Source: [codejunkie99/agentic-stack-desktop](https://github.com/codejunkie99/agentic-stack-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
