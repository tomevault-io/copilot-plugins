## mono

> See [[INIT]] for full project instructions.

# Claude Instructions

See [[INIT]] for full project instructions.

## Critical Rules

- **Before ANY Edit or Write tool call, verify you are NOT on main.** Create a task branch first (`git checkout -b task/<description>`). No exceptions.

- **When adding/modifying skills or prefixes**, update all related meta files:
  - New prefix → `meta - prefixes.md` AND `meta - file conventions.md` (dated filenames table)
  - New skill → Check if `INIT.md` sections reference it (Sprint Workflow, Snapshot Skills, etc.)
  - Plan template changes → `meta - coding project structure.md`

---
> Source: [ltOgt/mono](https://github.com/ltOgt/mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
