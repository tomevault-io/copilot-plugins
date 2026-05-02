## 00-workflow

> Core workflow rules. Apply to every agent session in this project.


# Core workflow rules

## Session start (mandatory)

At the start of every session, before writing any code:

1. Read `SCOPE.md` — understand the project, tech stack, and feature breakdown.
2. Read `TASKS.md` — find the current task state.
3. Read `memory/architecture.md` — understand existing module structure.
4. Read `memory/patterns.md` — use established patterns; do not invent duplicates.
5. Read `memory/decisions.md` — do not re-litigate past decisions.
6. Read `memory/stack-guidance.md` — use stack-specific senior defaults when it exists.
7. Skim every `.cursor/rules/*.mdc` file. They are all `alwaysApply: true` and
   cover workflow, verification, memory, sub-agents, design fidelity,
   security, observability, and concurrency. Do not apply only a subset.

If any of these files do not exist yet, skip it and continue.

## Trigger phrase recognition

Recognize and act on trigger phrases from `AGENTS.md`:
- `import docs` → run import-docs skill (docs → SCOPE + stack-guidance)
- `figma ingest` → run figma-plugin-ingest skill
- `reqops` → run reqops skill for target feature requirements
- `parse scope` → run parse-scope skill
- `delta scope` / `rescope` → run delta-scope skill when SCOPE changes after tasks exist
- `refresh mcp` → force `fetch-mcp` / `figma ingest` to ignore TTL and refetch
- `start build` → run orchestrate skill
- `resume build` → resume scheduler loop from current task states
- `qa only` → run QA skill on in-review tasks only
- `show status` → summarize TASKS.md counts

## Task discipline

- Work on one task at a time within a session.
- Never implement code outside the files listed in the current task block.
- Never invent features not described in `SCOPE.md`.
- Never mark a task `done` — only the QA agent does that.
- If something is unclear: mark the task `blocked`, write the specific question, surface it to the user. Do not guess.

## File discipline

- Always read a file before editing it.
- Always place new files in the location indicated by `memory/architecture.md`.
- Never delete files unless the task explicitly requires it.
- Never commit secrets, credentials, or `.env` values.

## Code quality (always)

- No `any` types unless absolutely unavoidable and commented with reason.
- No `TODO` comments left in completed code — if something is unfinished, the task is not done.
- No hardcoded values that belong in config or environment variables.
- Error paths must be handled — no silent swallows.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
