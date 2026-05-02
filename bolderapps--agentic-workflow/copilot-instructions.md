## 03-subagents

> Sub-agent roles, handoff protocol, and parallelism rules. Apply whenever the orchestrate skill is active.


# Sub-agent rules

## Roles summary

| Role | Count | Writes code? | Marks done? | Marks in-review? |
|------|-------|-------------|------------|-----------------|
| Orchestrator | 1 | No | No | No |
| Builder | Dynamic (1 per independent ready chain) | Yes | No | Yes |
| QA Agent | 1 shared | No (fix notes only) | Yes | No |

## Status transitions (only valid paths)

```
pending → in-progress     (Builder picks up task)
in-progress → in-review   (Builder finishes, verification passes)
in-progress → blocked     (Builder hits blocker)
in-review → done          (QA approves)
in-review → needs-fix     (QA rejects)
in-review → blocked       (QA rejects but attempt budget is exhausted)
needs-fix → in-progress   (Builder picks up fix)
blocked → pending         (blocker resolved, task returned to scheduler queue)
blocked → in-progress     (blocker resolved and original Builder resumes directly)
pending → obsolete        (only via delta-scope skill; archive, do not delete)
```

Any other transition is invalid. Agents must not skip steps.

## Parallelism rules

**Can run in parallel:**
- Tasks in different feature groups with no shared files
- Tasks with no dependency edge between them
- Tasks whose file sets do not overlap

**Must run sequentially (same Builder, in order):**
- Tasks that share any file
- Tasks where one `Depends on` the other
- Any task that depends on the scaffold task (scaffold must be `done` first)

**Orchestrator decides at `start build` time.** It reads all task blocks, builds a dependency graph, groups independent chains, and assigns one Builder per chain (`builder-1 ... builder-N`).

## File ownership

When the Orchestrator fans out tasks:
- Each Builder "owns" the files in its assigned tasks.
- No Builder may modify a file owned by another Builder's active task.
- If two tasks need to modify the same file, they must be in the same Builder's chain, run sequentially.
- Shared utility files (`memory/`, `types/`, `config/`) are written by whichever Builder creates them first, then read-only for others until that Builder's task is `done`.

## Builder handoff note format

When marking a task `in-review`, Builder writes a brief note in the task block:

```
- QA notes: Built POST /auth/login handler in src/api/auth.ts. Added LoginScreen in
  src/screens/Login.tsx using cached Figma context (memory/mcp-cache/auth-figma.md).
  Tests in src/tests/auth.test.ts cover happy path + 401 + network error.
  Lint and typecheck pass. Acceptance criteria self-checked.
```

## QA rejection note format

When marking a task `needs-fix`, QA writes specific notes:

```
- QA notes: REJECTED.
  1. src/api/auth.ts:34 — missing error handling for DB connection failure.
     Expected: catch block that returns 500 with { error: "internal", code: "DB_ERROR" }.
  2. src/tests/auth.test.ts — no test for 500 error path. Add one.
  3. Lint: unused import 'hashPassword' at src/api/auth.ts:2. Remove.
```

Vague notes like "tests are wrong" are not acceptable.

## Blocked task protocol

When a Builder marks a task `blocked`:
1. Write the specific reason: missing dependency, ambiguous spec, unreachable MCP URL, repeated test failure.
2. If it's an ambiguous spec: paste the exact sentence from `SCOPE.md` that is unclear and ask the specific question.
3. Orchestrator surfaces all `blocked` tasks to the user at the next check interval.
4. Do not guess. Do not invent a solution. Wait for human input.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
