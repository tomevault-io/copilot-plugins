## knowns

> Compatibility entrypoint for runtimes that auto-detect `GEMINI.md`.

# GEMINI

Compatibility entrypoint for runtimes that auto-detect `GEMINI.md`.

<!-- KNOWNS GUIDELINES START -->

@KNOWNS.md

**CRITICAL: You MUST read and follow `KNOWNS.md` in the repository root before doing any work. It is the canonical source of truth for all agent behavior in this project.**

## Canonical Guidance

- Knowns is the repository memory layer for humans and the AI-friendly working layer for agents.
- The source of truth for repo-level agent guidance is `KNOWNS.md`.
- Read `KNOWNS.md` first whenever the runtime supports reading repository files.
- If this file and `KNOWNS.md` differ, follow `KNOWNS.md`.

## Minimum Rules

- Use Knowns as the canonical system for tasks, docs, templates, and workflow state.
- Never manually edit Knowns-managed task or doc markdown.
- Search first, then read only relevant docs and code.
- Plan before implementation unless the user explicitly overrides that workflow.
- Validate before considering work complete.
- Use memory tools: `list_memories` at session start, `add_memory` after tasks for reusable knowledge, `add_working_memory` for session cache.

## Quick Reference

```bash
knowns doc list --plain               # List docs
knowns task list --plain              # List tasks
knowns task <id> --plain              # View task
knowns doc "<path>" --plain --smart  # View doc
knowns search "query" --plain        # Search docs/tasks
knowns guidelines --plain             # Full workflow reference
```

<!-- KNOWNS GUIDELINES END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/knowns-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/knowns-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
