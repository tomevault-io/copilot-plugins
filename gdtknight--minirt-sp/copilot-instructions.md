## minirt-sp

> When executing any speckit-related commands (`/speckit.*`), you MUST follow these rules:

# Claude Code Project Instructions

## Speckit Command Guidelines

When executing any speckit-related commands (`/speckit.*`), you MUST follow these rules:

### Mandatory Pre-Read Sequence

Before performing ANY speckit action, read these files in order:

1. `AGENTS.md` - Global agent rules (highest priority)
2. `memory/constitution.md` - Long-term project rules and invariants
3. `memory/context.md` - Current project state and constraints
4. Latest files under `spec/`
5. Latest files under `tasks/`

### SSOT Priority Order

If there is any conflict between sources, follow this priority (highest first):

1. AGENTS.md
2. memory/constitution.md
3. memory/context.md
4. spec/*
5. tasks/*
6. README and source files

### Task Discipline

- Never implement anything not specified in `spec/*.md`
- Never start work on anything not listed in `tasks/tasks.md`
- If a task is missing, add it to `tasks/tasks.md` before starting
- Task status must be one of: TODO, DOING, BLOCKED, DONE

### Change and Decision Policy

After completing any meaningful work:

1. Update `tasks/tasks.md` with the new status
2. Append a short entry to `memory/decisions.md` describing:
   - What changed
   - Why it changed
   - Any important trade-offs

### Ambiguity Handling

- If requirements are unclear or missing, do NOT guess
- Write a clear question into `tasks/questions.md`
- Continue only with confirmed information

### Output Rules

- Prefer minimal diffs and focused changes
- Do not modify unrelated files
- Do not duplicate SSOT information across multiple files
- Before creating or replacing templates, ask the user to confirm

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gdtknight) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
