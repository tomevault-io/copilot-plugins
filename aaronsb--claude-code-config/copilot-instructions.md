## claude-code-config

> ADR-driven workflow with GitHub-first collaboration.

# ADR-Driven Development Config

ADR-driven workflow with GitHub-first collaboration.

**Instructions are injected via hooks, not this file.**

Instructions are loaded at critical moments to maintain relevance:
- **SessionStart** - Fresh context when sessions begin
- **PreCompact** - Fresh context after compaction events

This approach ensures guidance stays active in the conversation window rather than being buried as distant system prompts.

See `hooks/ways/core.md` for the base guidance and `hooks/ways/*.md` for contextual instructions.

---
> Source: [aaronsb/claude-code-config](https://github.com/aaronsb/claude-code-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
