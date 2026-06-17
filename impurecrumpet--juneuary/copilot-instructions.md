## juneuary

> Skills harness for Cursor


# Skills Harness (Cursor)

Skills are in `.skills/_skills/`. The index is at `.skills/_index.md`.

## Rules

- Read `.skills/_index.md` at the start of any non-trivial task.
- Load a skill's full `SKILL.md` only when the task matches its triggers in the index.
- Never load skills preemptively.
- If a skill lists `dependencies`, load those skills before proceeding.
- **`.skills/_index.md` is the source of truth.** When you create, rename, or delete a skill, update the index in the same operation. Never leave the index out of sync with `.skills/_skills/`.
- If `.skills/` is missing from the repo, warn the user and do not invent skill content.
- **Subtree-vendored installs:** `.skills-harness/` is upstream-owned (updates via `git subtree pull`); use **harness-subtree** for kit maintenance.
- **Juneuary project skills** are prefixed `j-` (consumer-authored). Kit skills are unprefixed.

---
> Source: [ImpureCrumpet/Juneuary](https://github.com/ImpureCrumpet/Juneuary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
