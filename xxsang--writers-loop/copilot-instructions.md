## writers-loop

> This repository provides the `writers-loop` skill.

# Gemini Project Context

This repository provides the `writers-loop` skill.

Load `skills/writers-loop/SKILL.md` for substantial writing tasks that need planning, drafting, critique, revision, translation, style distillation, or local preference learning.

Rules for Gemini agents:

- Do not infer durable preferences from unreviewed model output.
- Separate current-task constraints from reusable preferences.
- Use local preference storage only after explicit opt-in.
- Preserve code, commands, paths, URLs, IDs, names, and markdown structure during translation unless the user says otherwise.
- Refer to `docs/writers-loop-complete-guide.md` for a fuller user-facing explanation.

---
> Source: [xxsang/writers-loop](https://github.com/xxsang/writers-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
