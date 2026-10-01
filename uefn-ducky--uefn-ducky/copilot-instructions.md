## project-folder-storage

> HARD — never store Ducky side-files in UEFN project except .ducky/**


# Project folder storage — `.ducky` only

**NEVER** create Ducky side-files inside the UEFN project folder except under
`.ducky/**` (tests, tasks).

| Allowed in project | Forbidden |
|--------------------|-----------|
| `.ducky/**` (JSON/tasks — never `.py`) | `Saved/DuckyCaptures`, `.uefn-ducky`, caches, temps, extra `*.py` / `*.pyc` |
| `Content/**` game content (Verse, assets) | Agent scratch, dumps, plugin junk |
| `Content/Python/init_unreal.py` only (Ducky-managed listener — **never delete**) | Any other `Content/Python/**` file |

**Put everything else here:**

- `%LOCALAPPDATA%/UEFN-Ducky/` (tool_captures, memory, diagnostics, plugins, …)
- OS temp when truly ephemeral

Captures / snips → AppData `tool_captures` only. Never mirror into the island.

---
> Source: [UEFN-Ducky/UEFN-Ducky](https://github.com/UEFN-Ducky/UEFN-Ducky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
