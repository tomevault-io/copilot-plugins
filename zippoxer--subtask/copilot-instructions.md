## subtask

> Parallel task management and orchestration for AI coding agents.

# Subtask

Parallel task management and orchestration for AI coding agents.

A lead agent dispatches work to parallel workers in isolated git worktrees, with context preservation and progress tracking.

---

## Philosophy

### The Problem

Without Subtask, a user managing multiple AI coding sessions must:
- Open multiple terminal tabs, each running Claude Code or Codex
- Mentally track what each session is working on
- Juggle progress, blockers, and dependencies in their head
- Re-explain context when sessions crash or expire
- Be the project manager while also trying to get work done

This cognitive load is exhausting. The user becomes a human orchestrator for AI workers.

### The Solution

Subtask shifts the cognitive burden from user to lead agent.

**Three roles:**
- **User**: Gives direction, makes decisions, approves work
- **Lead** (e.g., Claude Code): Orchestrates everything in between
- **Workers**: Execute tasks in parallel, isolated in git worktrees

The user provides direction. The lead handles everything else: understanding requirements, breaking work into tasks, dispatching to workers, tracking progress, reviewing output, iterating until it's right, merging when ready.

The lead is not a task dispatcher. The lead is a technical lead / project manager. We're building tools that enable this.

### Design Goals

Every feature should ask:

1. **Does this reduce cognitive load on the user?**
2. **Is this simple for the lead to use and understand?**

If a feature adds complexity for the user, it's probably wrong. If it confuses the lead, it's also wrong. Internal complexity is fine if the interface stays simple.

---

## Design Decisions

1. **Task-centric** — Task name is the primary identifier. Everything flows from it.
2. **Git-native** — Branches for isolation, worktrees for parallelism, standard merge workflow.
3. **File-based collaboration** — Task folder shared between lead and worker. PLAN.md for plans, PROGRESS.json for tracking. Files persist; sessions don't.
4. **Workspace opacity** — Lead never picks workspaces. Subtask assigns them.
5. **Context preservation** — Task folders are the portable, syncable unit. history.jsonl and `--follow-up` ensure nothing is lost when sessions crash. Copy anywhere, full context. Internal state and caches are local and rebuildable.
6. **Progress visibility** — Tool counts, timing, and PROGRESS.json let lead track workers without interrupting them.
7. **Guardrails over escape hatches** — Prevent footguns. Errors guide the lead on what to do next, not just fail. Dangerous operations require explicit intent.
8. **Local-first** — Operations use local git state. Lead controls when to sync with remote. No stale remote ref surprises.
9. **Event sourcing** — `history.jsonl` is the append-only source of truth. SQLite index is a derived projection for fast queries. If they diverge, history wins.

---

## Concepts

### Lead vs Worker

**Lead** (e.g., Claude Code, Codex, OpenCode):
- Runs `subtask` CLI commands
- Sees CLI output (`list`, `show`, errors)
- Drafts tasks, sends prompts, reviews work
- Reads task folders and history.jsonl

**Worker** (e.g., Codex, Claude):
- Receives prompts via harness
- Sees the repository and task folder
- Updates PROGRESS.json
- Never sees CLI output or other tasks

### Task

A named unit of work: `fix/epoch-boundary`
- Folder at `.subtask/tasks/<name>/` (with `/` escaped as `--`)
- Contains TASK.md (description), optional PLAN.md, PROGRESS.json
- Symlinked into workspace for lead/worker collaboration

### Workspace

Isolated git worktree where tasks execute. Created on-demand from a configured pool. Lead never picks workspaces—subtask assigns them.

### Harness

Worker backend that executes prompts. Supports `codex` and `claude`. Configured in `.subtask/config.json`.

### Workflow & Stages

Default: `doing → review → ready`

Planning workflows add a plan stage: `plan → implement → review → ready`
- `--workflow you-plan`: Lead drafts PLAN.md, worker reviews
- `--workflow they-plan`: Worker drafts PLAN.md, lead reviews

### Status & Transitions

**Task Status** (organizational, durable):
| Status | Meaning |
|--------|---------|
| `open` | Task is active |
| `merged` | Work merged into base branch |
| `closed` | Closed without merging |

Transitions:
- `open` → `merged` (via `merge`)
- `open` → `closed` (via `close`)
- `merged` → `open` (via `send`—revives to fix issues)
- `closed` → `open` (via `send`—revives with new workspace)

**Worker Status** (ephemeral, within an open task):
| Status | Meaning |
|--------|---------|
| `idle` | No worker activity yet |
| `running` | Worker currently executing |
| `replied` | Worker finished, awaiting follow-up |
| `error` | Last run failed |

Transitions:
- `idle` → `running` (via `send`)
- `running` → `replied` (worker finishes) or `error` (failure)
- `replied` → `running` (via `send`)
- `error` → `running` (via `send`)

Task status is what users care about. Worker status is operational detail. Workspace stays with task until closed/merged.

---

## Commands

| Command | Description |
|---------|-------------|
| `subtask install` | One-time global install + configuration wizard |
| `subtask config` | Edit user defaults or project overrides |
| `subtask draft <task>` | Create a task without running it |
| `subtask send <task>` | Send a message (starts or resumes task) |
| `subtask stage <task> <stage>` | Advance workflow stage |
| `subtask list` | Show all tasks and workspaces |
| `subtask show <task>` | Task details, progress, diff stats |
| `subtask merge <task> -m "..."` | Squash-merge into base branch, close |
| `subtask close <task>` | Close without merging (`--abandon` discards changes) |
| `subtask workspace <task>` | Print workspace path |
| `subtask ask "..."` | Quick question (no task, runs in cwd) |
| `subtask interrupt <task>` | Gracefully stop a running worker |
| `subtask log <task>` | Show conversation and lifecycle events |
| `subtask trace <task>` | Debug session logs (tool calls, timing) |

---

## Storage

```
~/.subtask/
├── config.json                              # global defaults (from install/config)
├── workspaces/<escaped-git-root>--<id>/     # worktrees (created on demand)
└── projects/<escaped-git-root>/             # per-project runtime state (machine-local)
    ├── internal/                            # session IDs, workspace assignments, locks
    └── index.db                             # SQLite cache (rebuildable)

<repo>/.subtask/
├── config.json                              # optional per-project overrides
└── tasks/<name>/                            # task folder (portable, syncable)
    ├── TASK.md                              # description + schema version in frontmatter
    ├── PLAN.md                              # optional plan
    ├── PROGRESS.json                        # worker progress tracking
    └── history.jsonl                        # source of truth: messages + lifecycle events
```

### Portability Contract

**Task folder** (`.subtask/tasks/<name>/`) is portable and syncable:
- Copy to another machine, sync via git/syncthing, back up
- Contains everything needed to understand and resume the task
- `history.jsonl` is the source of truth for task status, stage, messages

**Internal folder** (`.subtask/internal/<name>/`) is runtime-only:
- Workspace paths, session IDs—machine-specific
- Rebuilt automatically when needed
- Never sync or back up

**SQLite index** is a local cache:
- Fast queries for `list` and TUI
- Rebuilt from task folders if missing
- Never sync—it's a projection, not source data

---

## Project Layout

```
.
├── cmd/subtask/          # CLI commands and main.go entry point
├── task/                 # Task/State structs, paths, locking, progress
│   ├── gather/           # Shared data layer (used by CLI and TUI)
│   ├── history/          # history.jsonl: append, read, tail
│   ├── index/            # SQLite index for fast list/TUI queries
│   ├── migrate/          # Schema migrations (legacy → current)
│   └── ops/              # Task operations (merge, close)
├── tui/                  # Interactive TUI (Bubble Tea)
├── workspace/            # Workspace pool allocation
├── harness/              # Worker backends (Codex, Claude) and prompt building
├── git/                  # Git operations (branches, merge, diff, worktrees)
├── render/               # CLI output formatting (TTY detection, colors, tables)
├── workflow/             # Workflow stage templates (YAML, embedded)
├── logs/                 # Session log parsing and formatting
├── testutil/             # Test helpers (isolated environments)
├── e2e/                  # Integration tests
├── skill/
│   └── SKILL.md          # Embedded skill source (synced at runtime)
└── plugin/
    ├── .claude-plugin/
    └── setup.md
```

---

## Working on This Codebase

### Build & Test

```bash
go build ./cmd/subtask
go test ./... -short      # fast
go test ./...             # includes e2e
```

### Adding a Command

1. Create `cmd/subtask/<cmd>.go`
2. Define struct with kong tags
3. Implement `Run() error`
4. Add to CLI struct in `main.go`

### Adding a Harness

1. Create `harness/<name>.go`
2. Implement `Harness` interface
3. Register in `harness.New()`

---
> Source: [zippoxer/subtask](https://github.com/zippoxer/subtask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
