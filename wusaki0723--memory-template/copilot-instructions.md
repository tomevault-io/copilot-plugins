## memory-template

> This repository is a private Markdown memory vault with GitHub Issues as the raw memory stream.

# Memory Vault Instructions

This repository is a private Markdown memory vault with GitHub Issues as the raw memory stream.

## Read Order

1. Read this file first.
2. Read `memory-index.md`.
3. Read only the files or issues needed for the current request.
4. Do not scan the whole repository unless the user asks for an audit, migration, or re-index.

## Repository Memory

Files in the repository are distilled memory: stable facts, current relationship state, boundaries, style preferences, and reusable prompts.

## Issue Memory

Issues are raw memory: logs, fragments, daily notes, emotions, scenes, drafts, and items waiting to be organized.

The issue form's `Memory type` field is the source of truth. If matching labels are available, also add one `type/*` label such as `type/style` or `type/event` after the issue is submitted. GitHub issue forms cannot automatically pick a dynamic label from a dropdown value.

Use issue labels and GitHub search before reading issue bodies. Prefer narrow searches such as:

```text
is:issue is:open label:status/raw label:type/event updated:>=YYYY-MM-DD
```

If a copied private repository does not have type labels yet, search by `label:status/raw` plus words from the issue form until labels are created.

## Claude Code Project Tools

- Project skill: `.claude/skills/memory-chat/`
- Project MCP: `.mcp.json` defines the local `memory_vault` server.
- Project hooks: `.claude/settings.json` injects `style/voice-and-tone.md` on each `UserPromptSubmit` and writes compact event archives to `archive/hook-events/`.

Treat hook archives and MCP raw-memory files as raw memory. Distill useful long-term items into the normal vault files before relying on them as stable truth.

In cloud Claude Code, generated archive files survive only if they are committed and pushed.

## Truth Rules

- Do not invent memories.
- If a detail is missing, say it is not in the vault.
- If old and new memories conflict, cite both and ask which one to keep.
- Treat `imports/` and raw issues as unconfirmed until distilled.

## Update Rules

- Make small edits.
- Preserve old memories unless the user asks to delete them.
- Put stable facts in `people/` or `relationship/`.
- Put dated memories in `events/`.
- Put style preferences in `style/`.
- Update `memory-index.md` when adding or moving files.
- Append `changelog.md` when repository memory changes.
- When distilling issues, comment on the source issue with the destination file path and change its status label if labels are available.

---
> Source: [wusaki0723/memory-template](https://github.com/wusaki0723/memory-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
