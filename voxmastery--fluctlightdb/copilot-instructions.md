## fluctlightdb

> This monorepo uses FluctlightDB (`FluctlightDB`) for durable agent memory:

## FluctlightDB (project memory)

This monorepo uses FluctlightDB (`FluctlightDB`) for durable agent memory:

- Hub: `.fluctlight/project/` (shared decisions + handoffs)
- Spokes: `.fluctlight/agents/{cursor,claude,codex}/`

Use `from fluctlightdb import connect_project` and call `session_context()`, `recall()`, `remember()`, and `handoff()` when switching agents or resuming work. See `.claude/skills/fluctlight-memory/SKILL.md`.

---
> Source: [voxmastery/FluctlightDB](https://github.com/voxmastery/FluctlightDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
