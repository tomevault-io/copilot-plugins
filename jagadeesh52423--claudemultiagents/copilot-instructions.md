## claudemultiagents

> This project uses a team of specialized AI subagents — always parallel, always as a team. No sequential mode.

# Multi-Agent Software Development System

This project uses a team of specialized AI subagents — always parallel, always as a team. No sequential mode.

## Agent Catalog

| Agent | Role | When to use |
|-------|------|-------------|
| `architect-designer` | Creates software designs from requirements | COMPLEX/EPIC tasks only |
| `architect-reviewer` | Reviews and critiques designs | COMPLEX/EPIC tasks only |
| `developer-coder` | Writes production code from designs | All tasks |
| `developer-reviewer` | Reviews code for quality and bugs | SIMPLE and above |
| `tester` | Tests code by actually running it | MODERATE and above |
| `documenter` | Creates project documentation | EPIC tasks only |

## FULL_TASKS.md — Persistent Task Tracker (Mini Jira)

### Session Start Protocol (MANDATORY)

**Every session MUST begin by checking for `FULL_TASKS.md` in the project root.**

1. **If `FULL_TASKS.md` exists**: Read it, display a compact board summary, and ask the user what to work on.
2. **If `FULL_TASKS.md` does NOT exist**: After understanding the user's request, create it.

### Format

```markdown
# Project Task Board
> Last updated: {date} | Session: {n}

## Legend
- `TODO` — Not started
- `IN_PROGRESS` — Currently being worked on
- `IN_REVIEW` — Code written, under review
- `TESTING` — Under testing
- `DONE` — Completed and verified
- `BLOCKED` — Waiting on something

## Agent Registry
| Agent ID | Name | Domain | Tasks |
|----------|------|--------|-------|
| {id} | coder-auth | Auth, sessions | TASK-001, TASK-005 |
| {id} | coder-payments | Billing | TASK-002, TASK-006 |

## Tasks

### TASK-001: {Title}
- **Status**: {status}
- **Priority**: P0 | P1 | P2 | P3
- **Complexity**: TRIVIAL | SIMPLE | MODERATE | COMPLEX | EPIC
- **Agents Needed**: {determined by complexity}
- **Assigned To**: {agent or unassigned}
- **Blocked By**: {TASK-XXX or none}
- **Description**: {what needs to be done}
- **Acceptance Criteria**:
  - [ ] {criterion}
- **Notes**: {context}
- **History**:
  - {date}: {event}
```

### Lifecycle Rules
- Break user work into discrete TASK-XXX items
- Update status before and after each phase
- Add History entry on every change
- Mark acceptance criteria as checked on completion
- FULL_TASKS.md persists across sessions

## Dynamic Agent Selection

**Only spawn agents the task actually needs.**

| Complexity | Agents | Use When |
|-----------|--------|----------|
| **TRIVIAL** | `developer-coder` | Typo, config, single-line fix |
| **SIMPLE** | `developer-coder` + `developer-reviewer` | Small bug fix, 1-2 file change |
| **MODERATE** | `developer-coder` + `developer-reviewer` + `tester` | New endpoint, enhancement, 2-5 files |
| **COMPLEX** | `architect-designer` + `architect-reviewer` + `developer-coder` + `developer-reviewer` + `tester` | New feature/module, 5+ files |
| **EPIC** | All 6 agents (+ multiple streams) | New project, major system change |

## Workforce Scaling — Hire and Fire

**Scale agent count based on workload. More independent tasks = more parallel agents.**

### Scaling Rules
- **N independent tasks → up to N coders** (capped at 6)
- **Scale reviewers**: 1 reviewer per 2-3 coders
- **Scale testers**: 1 tester per independent stream
- **Architects stay lean**: 1 designer + 1 reviewer (design before scaling)
- **Fire when done**: Shut down agents once their tasks are DONE

### Naming: `coder-auth`, `coder-payments`, `reviewer-backend`, `tester-api` (by domain)

### Scaling Table
| TODO Tasks | Coders | Reviewers | Testers |
|-----------|--------|-----------|---------|
| 1 | 1 | 1 | 1 |
| 2-3 | 2-3 | 1-2 | 1-2 |
| 4-6 | 4-6 | 2-3 | 2-3 |
| 7+ | Cap 6 | Cap 3 | Cap 3 |

## Task Affinity — Same Developer, Related Tasks

**Assign related tasks to the same agent. Resume > fresh spawn for domain continuity.**

### Rules
1. **Same module → same agent**: All auth tasks → `coder-auth`
2. **Bug fix for code agent wrote → resume that agent**
3. **Reviewer who reviewed → same reviewer for re-review**
4. **Tester who tested → same tester for regression**

### Task Grouping Flow
1. Group TODO tasks by domain (files they touch)
2. Assign each group to one agent (affinity)
3. Order tasks within group by dependency
4. Spawn agents per group — groups run in PARALLEL, tasks within group handled by same agent

## Execution — Always Parallel, Always Teams

**Every execution uses TeamCreate. No sequential mode. No exceptions.**

### Announce Workforce Plan (MANDATORY)

Multi-task:
```
📋 Workforce Plan
Tasks: {N} TODO ({M} domains)
Staffing: {coders} coders, {reviewers} reviewers, {testers} testers
Affinity Groups:
  - {Domain} ({agent-name}): TASK-XXX, TASK-YYY
Workflow: Design → Parallel Code → Parallel Review → Parallel Test
```

Single task:
```
📋 Workforce Plan
Task: TASK-XXX — {title}
Complexity: {level}
Staffing: {agents}
Workflow: {phases} (all in team)
```

### Workflow (All Complexities)

1. **TeamCreate** → create team
2. **TaskCreate** → create tasks with `blockedBy` dependencies per phase
3. **Spawn ALL needed agents in one message** — each joins team, blocked agents wait
4. **Orchestrate** — unblock tasks, handle revisions, forward questions, manage iterations
5. **Fire agents** as their domain completes
6. **TeamDelete** → cleanup when all done
7. **Update FULL_TASKS.md** → mark DONE, summarize to user

### Parallel Safety Rules
- NEVER let two agents write to the same file
- Each agent/domain has its own file list
- Shared files → assigned to ONE agent only
- If two agents need same file → assign to one, other waits

### Even Single Tasks Use Teams
A single TRIVIAL task: `TeamCreate` → spawn coder → complete → `TeamDelete`. Uniform workflow, no special cases.

## Session Management (Agent Resume)

| Scenario | Action |
|----------|--------|
| Revision loop (same agent, same task) | **Resume** — feedback delta only |
| Related task, same domain (affinity) | **Resume** — agent knows the domain, just describe new task |
| Unrelated new task | **Fresh** — different context |
| Different agent, same workflow | **Fresh** — pass artifact paths |

## Shared Artifacts
```
FULL_TASKS.md      ← team lead maintains
DESIGN.md          ← architect-designer writes
DESIGN_REVIEW.md   ← architect-reviewer writes
CODE_REVIEW.md     ← developer-reviewer writes
TEST_REPORT.md     ← tester writes
FEATURE_DOCS.md    ← documenter writes
```

## Subagent Model (CRITICAL — MUST FOLLOW)

**Every subagent MUST be spawned with `--model anthropic.claude-opus-4-6-v1`.**

This is mandatory. The LiteLLM proxy only accepts this exact model ID. Any alias causes 401.

### How to enforce:
1. **Agent definitions**: Frontmatter has `model: anthropic.claude-opus-4-6-v1` — already set
2. **Agent tool calls**: MUST pass `model: "anthropic.claude-opus-4-6-v1"` explicitly. Do NOT omit. Do NOT rely on inheritance.
3. **NEVER use aliases**: No `model: "opus"`, `model: "sonnet"`, or `model: "haiku"`
4. **Dynamic/ad-hoc agents**: Same rule — always `--model anthropic.claude-opus-4-6-v1`

### Correct:
```
Agent(subagent_type: "developer-coder", model: "anthropic.claude-opus-4-6-v1", ...)
```
### WRONG (401 error):
```
Agent(subagent_type: "developer-coder", model: "opus", ...)
```

## Rules
- Delegate to agents via Agent tool — don't do their work
- **Always use TeamCreate** — every execution is a team, no exceptions
- Read output files after each agent to check results
- **Update FULL_TASKS.md** on every status change
- Max iterations: 3 for design/code, 2 for testing
- **Resume** for revision loops and affinity tasks
- **Scale agents with workload** — N independent tasks = N parallel coders
- **Use task affinity** — related tasks → same agent, resume for continuity
- **Fire idle agents** — shut down when domain tasks are DONE
- **Track Agent Registry** in FULL_TASKS.md for resume mapping
- Tester MUST run code — ask user to start server
- Reviewer MUST trace values across files

---
> Source: [jagadeesh52423/claudeMultiAgents](https://github.com/jagadeesh52423/claudeMultiAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
