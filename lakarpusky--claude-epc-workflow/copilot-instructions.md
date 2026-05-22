## claude-epc-workflow

> Personal defaults for Claude Code across all projects. Project-specific guidance lives in each project's own `CLAUDE.md` (see `~/.claude/templates/PROJECT-CLAUDE.md`).

# CLAUDE.md

Personal defaults for Claude Code across all projects. Project-specific guidance lives in each project's own `CLAUDE.md` (see `~/.claude/templates/PROJECT-CLAUDE.md`).

## Direct-mode rules

These apply when working without `/epc`. The agent system has its own coordination protocols defined in `~/.claude/agents/` and `~/.claude/commands/epc.md`.

- Modify only files directly related to the request. If other files need changes, explain why and wait for approval.
- When asked to fix staged files, run `git diff --cached --name-only` first and scope work to that exact list.
- Never `git add .` without confirming the file list.
- Never commit unless explicitly asked.
- Never run `git reset --hard` unless explicitly instructed. Use `git reset --soft HEAD~N` or `git stash` to preserve changes.

## Debugging

- Trace data/prop/event flow end-to-end before making any change. No speculative fixes.
- Identify entry point → follow chain of calls → explain root cause with file paths and line numbers BEFORE proposing edits.
- Exhaust automated verification first (run dev server, check console, inspect network). Only ask the user for environment-specific issues that can't be reproduced.

## Bash hygiene

- Don't pipe output through `head | tail | less | more` — causes buffering issues.
- Use command-specific flags (`git log -n 10`) instead of pipes when limiting output.

## When in doubt

- The `karpathy-guidelines` skill loads automatically for code-writing tasks. It encodes: surface assumptions before coding, simplicity first, surgical changes, goal-driven verification.
- For complex coding tasks, prefer planning mode before implementation.
- After any user correction, note the pattern in your memory so the same mistake doesn't recur.

---
> Source: [lakarpusky/claude-epc-workflow](https://github.com/lakarpusky/claude-epc-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
