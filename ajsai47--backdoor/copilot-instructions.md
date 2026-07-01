## backdoor

> <!-- Fill in your project here. Be specific — the model has no prior knowledge of your codebase. -->

# Project Context

<!-- Fill in your project here. Be specific — the model has no prior knowledge of your codebase. -->
<!-- Example: "This is a FastAPI service that processes payments. Main entry point is src/main.py." -->

---

# Tool Use

- **Always Read before Edit.** Never edit a file you haven't read in this session. The Edit tool will fail if content doesn't match exactly — read first, then edit.
- **Prefer dedicated tools over Bash for file ops.** Use Read, Edit, Write, and Glob directly. Only use Bash when the task genuinely requires a shell (running tests, git commands, build steps).
- **Use Bash for searches only when Grep/Glob won't do.** For pattern searches across files, prefer the Grep tool. Use `find` in Bash only when you need filesystem metadata.
- **Don't use `cat`, `head`, `tail`, or `echo` in Bash.** Use the Read tool instead.
- **Quote all file paths with spaces** in Bash commands.
- **Sub-agents (Agent tool) are for isolated subtasks only.** Spawn one if a task is self-contained and doesn't need shared context. Don't use it to parallelize everything.

# Task Discipline

- Use TodoWrite at the start of any multi-step task. List every step you plan to take.
- Mark each todo complete immediately after finishing it — not at the end.
- One step at a time. Don't batch multiple unrelated changes into a single edit.
- If a task grows in scope mid-way, stop and surface it. Don't silently expand.

# Code Quality

- No unnecessary comments. Code should be self-explanatory; only comment non-obvious decisions.
- No speculative abstractions. Build what's needed now, not what might be needed later.
- No half-finished work. If you start a change, complete it or revert it — never leave code in a broken state.
- Don't rewrite working code in a different style. Fix what's broken; leave what isn't.
- Match the existing style of each file. Indentation, naming, spacing — follow what's already there.

# Confirmation Required

Stop and ask before running any of these:

- `git reset --hard` or `git clean -f`
- `git push --force` or `git push -f`
- `rm -rf` on any directory
- Dropping or truncating a database table
- Any irreversible operation on production data

# Response Style

- Short and direct. No preamble, no trailing summaries.
- While working: one sentence max per update ("Reading config file.", "Editing handler.", "Done.").
- Don't explain what you're about to do and then do it — just do it.
- Don't recap completed steps at the end of a response.
- If you hit an ambiguity that blocks progress, ask one specific question. Don't list every possible interpretation.

# Git

- Never commit unless explicitly asked.
- Never amend a previous commit — create a new one.
- Never skip hooks (`--no-verify`).
- Commit messages: imperative mood, under 72 chars, no period at the end.

---
> Source: [ajsai47/backdoor](https://github.com/ajsai47/backdoor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
