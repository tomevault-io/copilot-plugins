## scenes-generator

> **RULE: Before doing anything else in a session, read `.max/project-context.json` from the workspace root.**

# MAx Project Context

**RULE: Before doing anything else in a session, read `.max/project-context.json` from the workspace root.**

This file is pre-scanned by the MAx extension on workspace open. It contains everything needed to understand the project. Do NOT scan project files manually when this file exists — use it directly.

When asked anything about the project (what it is, what stack, what's been built, current state, etc.) — answer from `.max/project-context.json` immediately without exploring the filesystem.

Only scan files manually if:
- `.max/project-context.json` does not exist
- The user explicitly asks to re-scan or refresh context
- The user asks about something not covered in the context file (e.g. specific file contents)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danilonovaisv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
