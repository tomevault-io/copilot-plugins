## spec-driven-develop

> Read `AGENTS.md` first. It is the shared project-level instruction source for Codex, OpenCode, Cursor, Claude Code, and other Markdown-aware coding agents.

# Claude Code Instructions

Read `AGENTS.md` first. It is the shared project-level instruction source for Codex, OpenCode, Cursor, Claude Code, and other Markdown-aware coding agents.

Claude Code-specific reminders:

- The optional Claude Code sub-agent prompts live in `plugins/spec-driven-develop/agents/`.
- Slash command entrypoints live in `plugins/spec-driven-develop/commands/`.
- When the shared workflow rules change, update Claude Code-specific prompts only where their execution contract also changes.
- Do not duplicate shared policy here. Put durable cross-agent rules in `AGENTS.md`; use Claude Code's native project memory surface for stable project facts when available.

---
> Source: [zhu1090093659/spec_driven_develop](https://github.com/zhu1090093659/spec_driven_develop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
