## treejar

> > **IMPORTANT**: This file overrides default Claude Code behavior. Follow strictly.

# Agent Orchestration Rules

> **IMPORTANT**: This file overrides default Claude Code behavior. Follow strictly.

## Main Pattern: You Are The Orchestrator

### Core Rules

**1. GATHER CONTEXT FIRST (MANDATORY)**

- Read existing code, search patterns, check recent commits
- NEVER delegate or implement blindly

**2. DELEGATE TO SUBAGENTS**

- Provide complete context (code, paths, patterns)
- **NEVER TRUST SUBAGENT REPORTS** — always verify yourself:
  - Read modified files (`Read` tool)
  - Run type-check (`pnpm type-check`)
  - Run build if needed (`pnpm build`)
  - Check for regressions
- Re-delegate if incorrect

**3. EXECUTE DIRECTLY** — Only for: single-line fixes, simple imports, minimal configs

**4. TRACK PROGRESS** — TodoWrite: in_progress BEFORE, completed AFTER verification

**5. COMMIT** — `/push patch` after each task

**6. EXECUTION PATTERN**

```
1. Read task → 2. Gather context → 3. Delegate/execute
4. VERIFY (never skip) → 5. Re-delegate if needed
6. TodoWrite completed → 7. /push patch → 8. Next task
```

**7. CONTRADICTIONS** — Gather context, analyze patterns. Ask user only if truly ambiguous (~10%).

**8. TYPESCRIPT ERRORS** — Re-delegate to same agent OR `typescript-types-specialist`

**9. /push — NEVER DISCARD CHANGES**

- **FORBIDDEN**: `git reset`, `git checkout --`, `git stash` during `/push`
- **ALWAYS** commit all uncommitted changes or ASK user first

---

## Task Management with Beads

> Constitution v1.2.0: All work MUST be tracked in Beads.

### Session Workflow

```bash
# SESSION START (auto via hooks)
# bd prime runs automatically → injects context

# FIND WORK
bd ready                          # Available tasks (no blockers)
bd ready --label frontend         # Only frontend tasks
bd list --unlocked                # Tasks not locked by other terminals
bd show <id>                      # Task details

# WORK
bd update <id> --status in_progress   # Acquires exclusive lock
# ... do the work ...
bd close <id> --reason "Done"         # Releases lock

# SESSION END
git add . && git commit -m "..." && git push
```

### Multi-Terminal Work

When working in multiple terminals simultaneously:

- Each terminal acquires **exclusive lock** via `bd update --status in_progress`
- Lock auto-releases after 30min inactivity
- **Rule**: Each terminal works on DIFFERENT issues
- Find unlocked: `bd list --unlocked`

### Task Types

| Work Type            | Tool            | Command                                              |
| -------------------- | --------------- | ---------------------------------------------------- |
| Big feature (>1 day) | Bonded Pipeline | `bd mol bond bigfeature-pipeline`                    |
| Small feature        | Beads           | `bd create -t feature --files path/to/file.tsx`      |
| Bug fix              | Beads           | `bd create -t bug`                                   |
| Tech debt            | Beads           | `bd create -t chore`                                 |
| Exploration          | Beads wisp      | `bd mol wisp exploration`                            |
| Code review          | Patrol          | `bd patrol run code-review --vars "scope=X,topic=Y"` |
| Health check         | Patrol          | `bd patrol run health-check`                         |

### Automation

- **Daemon auto-sync**: Enabled (auto-commit, auto-push, auto-pull for beads)
- **Hooks**: Stop → `bd sync`
- **Directory Labels**: Auto-assigned based on `--files` path (see config.yaml)
- **Exclusive Lock**: Prevents conflicts in multi-terminal work

**Emergent work**: `bd create "Issue" -t bug --deps discovered-from:<current-id>`

**Guide**: `.claude/docs/beads-quickstart.md`

---

## Project Conventions

**File Organization**:

- Agents: `.claude/agents/{domain}/{orchestrators|workers}/`
- Commands: `.claude/commands/`
- Skills: `.claude/skills/{name}/SKILL.md`
- Temp: `.tmp/current/` (gitignored)
- Reports: `docs/reports/{domain}/{YYYY-MM}/`

**Code Standards**: Type-check + build + lint must pass before commit. No hardcoded credentials.

**Agent Selection**:

- Worker: Plan file specifies `nextAgent` (health workflows only)
- Skill: Reusable utility, no state, <100 lines

**MCP Config**:

- `.mcp.json`: All servers with auto-optimization
- Includes: context7, sequential-thinking, supabase, playwright, shadcn

---

## External Documentation (Context7)

**Before implementing** with external libraries, query Context7 for latest docs:

```
mcp__context7__resolve-library-id → mcp__context7__query-docs
```


**Label-based routing**: Use `bd ready --label X` to find tasks for specific subagent.

**Rule**: For complex tasks, ALWAYS consider delegation. Verify result yourself.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maslennikov-ig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
