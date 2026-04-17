## scriptorium

> Rules for working on Scriptorium-based technical book projects


# Scriptorium — Windsurf Rules

This project uses the **Scriptorium** multi-agent framework for writing technical books
about open-source source code.

## Core Principle

Agents are stateless; the file system is stateful. Always read `checkpoint.md` at the
start of a session to understand current progress.

## File Conventions

- `agents/*.md` — Agent specification files. **Read-only.** Never modify these.
- `framework/*.md` — Framework documentation. **Read-only.**
- `templates/*.md` — Template files with `{{变量名}}` placeholders. Fill before use.
- `source-map.md`, `outline.md`, `style-guide.md` — Shared state. Fill in Phase 1.
- `checkpoint.md`, `audit-log.md` — Progress tracking. Update after each agent run.
- `research/` — Output from Researcher agent runs (Phase 2).
- `chapters/` — Output from Writer agent runs (Phase 3).
- `reviews/` — Output from R1/R2/R3 agent runs (Phase 4).

## Workflow

When asked to work on a book task, follow this pattern:

1. Read `checkpoint.md` to find the next incomplete task
2. Read the relevant agent spec in `agents/`
3. Load exactly the file pointers listed in that spec
4. Execute the agent's task
5. Write output to the path specified
6. Update `checkpoint.md` and `audit-log.md`

## Full Context

See `CLAUDE.md` for the complete project context and agent roster.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lordmos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
