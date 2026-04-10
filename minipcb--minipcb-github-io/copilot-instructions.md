## minipcb-github-io

> Instructions for Codex or other agents working in this repo.

# AGENTS.md

## Purpose
Instructions for Codex or other agents working in this repo.

## Priorities
- Preserve existing behavior unless requested.
- Prefer minimal, targeted edits.
- Keep changes consistent with existing style.

## Workflow
- Read relevant files before editing.
- Use the least invasive change that solves the task.
- Mention any assumptions.
- For any code/content change, append a log record to `agents_log.jsonl` and add a short summary row to `agents_log.md`.
- Every new log record must include per-file edit statements: file name plus 1+ concise statements of what was changed in that file.
- Treat `agents_log.jsonl` as append-only history; never rewrite old records (add a new correction/supersede record instead).

## Tools
- Prefer `rg` for search.
- Use `apply_patch` for small edits.

## Testing
- Run relevant tests if available.
- If not run, say so.

## Notes
- Avoid touching files not needed for the task.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miniPCB)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/miniPCB)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
