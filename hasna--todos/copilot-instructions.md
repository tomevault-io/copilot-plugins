## todos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Recommended Agent Workflow

> Full cross-tool workflow template: `configs show agent-workflow-template` (requires @hasna/configs)

When starting a work session as an AI agent:

```bash
# 1. Register yourself and claim a task
todos claim <your-agent-name>              # Atomic: find + lock + start best pending task
todos status                               # See full project health snapshot

# 2. Work on the task
todos log-progress <id> "Investigating..." # Record intermediate state (if OPE-00031 landed)

# 3. Complete with evidence
todos done <id> \
  --attach-ids <attachment-id> \           # Link uploaded evidence files
  --commit-hash <hash> \                   # Git commit reference
  --notes "All tests pass"                 # Completion notes

# 4. If you fail the task
todos fail <id> --reason "Auth bug in middleware" --retry  # Auto-creates retry copy
```

**Agent coordination:**
- `todos next` — what should I work on next?
- `todos active` — what are other agents working on right now?
- `todos stale` — are there any abandoned tasks to recover?
- `GET /api/tasks/stream?agent_id=<name>` — subscribe to real-time task events (SSE)

**MCP session start (one call instead of four):**
```
bootstrap(agent_id: "your-id")   # Returns: your active task, next claimable, health summary + as_of timestamp
```

**MCP incremental polling (preferred over re-fetching everything):**
```
# 1. Save as_of timestamp from bootstrap/get_status/get_context response
# 2. On next check, only fetch what changed:
get_tasks_changed_since(since: "<as_of timestamp>")   # Only returns tasks modified since last check
```

**MCP profile for minimal token cost:**
```
TODOS_PROFILE=minimal todos-mcp  # 11 tools: bootstrap, claim, complete, fail, status, get_context, get_task, start, add_comment, get_next_task, get_tasks_changed_since
TODOS_PROFILE=standard todos-mcp # ~47 tools (default for most workflows)
TODOS_PROFILE=full todos-mcp     # All 60+ tools
```

## Commands

```bash
bun install                       # Install dependencies
bun test                          # Run all 651 tests
bun test src/db/tasks.test.ts     # Run a single test file
bun run typecheck                 # TypeScript type checking (tsc --noEmit)
bun run build                     # Build all three entry points to dist/
bun run dev:cli                   # Run CLI in dev mode (bun run src/cli/index.tsx)
bun run dev:mcp                   # Run MCP server in dev mode
```

## Architecture

Universal task management system with three surfaces sharing a common SQLite database layer:

```
src/types/     → TypeScript types, enums, and custom error classes
src/db/        → SQLite data layer
                 database.ts   — schema, migrations, singleton, helpers
                 tasks.ts      — task CRUD, locking, dependencies
                 projects.ts   — project CRUD, prefix generation, counter
                 agents.ts     — agent registration (8-char UUID identity)
                 task-lists.ts — task list containers
                 plans.ts      — plan CRUD
                 comments.ts   — task comments
                 sessions.ts   — agent session tracking
src/lib/       → Business logic
                 search.ts     — full-text search across task fields
                 sync.ts       — bidirectional sync with agent task lists
                 config.ts     — config loading (~/.todos/config.json)
                 claude-tasks.ts — Claude Code task list sync
                 agent-tasks.ts  — Other agent task list sync
src/cli/       → Commander.js CLI (index.tsx) + React/Ink TUI components (components/)
src/mcp/       → MCP server with stdio transport (index.ts) — 29 tools, 4 resources
src/index.ts   → Library re-exports for external consumers (@hasna/todos)
```

All three surfaces (CLI, MCP server, library) call directly into `src/db/` functions — there is no intermediate service layer. The database module uses a singleton pattern via `getDatabase()`.

### Build Outputs

Three separate `bun build` invocations produce independent entry points:
- `dist/cli/index.js` — CLI executable (`todos` bin)
- `dist/mcp/index.js` — MCP server executable (`todos-mcp` bin)
- `dist/index.js` — Library entry point with `dist/index.d.ts` types

## Data Model

### Core Entities

| Entity | Key Fields | Notes |
|--------|-----------|-------|
| **Task** | id, short_id, title, status, priority, project_id, task_list_id, plan_id, parent_id, agent_id, assigned_to, version | short_id auto-generated from project prefix (e.g., "APP-00001") |
| **Project** | id, name, path (UNIQUE), task_prefix, task_counter | Prefix auto-generated from name, counter increments per task |
| **TaskList** | id, project_id, slug, name | Slug unique per project. Containers/folders for tasks |
| **Agent** | id (8-char), name (UNIQUE) | Registered via `init` command. Idempotent by name |
| **Plan** | id, project_id, name, status | Execution groups (active/completed/archived) |
| **TaskComment** | id, task_id, content, agent_id | Notes on tasks |
| **Session** | id, agent_id, project_id, metadata | Agent session tracking |

### Relationships

- Task → Project (FK, ON DELETE SET NULL)
- Task → TaskList (FK, ON DELETE SET NULL)
- Task → Plan (FK, ON DELETE SET NULL)
- Task → Task (parent_id, ON DELETE CASCADE — subtasks deleted with parent)
- TaskList → Project (FK, ON DELETE SET NULL)
- Plan → Project (FK, ON DELETE CASCADE)
- TaskDependency: task_id depends_on task_id (no self-deps, cycle detection via BFS)

## Key Patterns

- **Task short IDs**: When a task is created with a `project_id`, the project's `task_prefix` and `task_counter` generate a short_id (e.g., "APP-00001") that's prepended to the title. Projects without a prefix (created before migration 6) get one auto-assigned via `ensureProject()`.
- **Agent registration**: `registerAgent()` is idempotent — same name returns existing agent with updated `last_seen_at`. IDs are 8 characters from `crypto.randomUUID().slice(0, 8)`.
- **Optimistic locking**: Every task has a `version` integer. Updates require passing the current version; the UPDATE query includes `WHERE version = ?` and increments atomically. Mismatch throws `VersionConflictError`.
- **Exclusive task locking**: Tasks can be locked by an agent (`locked_by`/`locked_at`). Locks auto-expire after 30 minutes. Re-locking by the same agent is idempotent. Expired locks can be taken over by any agent.
- **Partial IDs**: All surfaces accept first 8+ characters of UUIDs via `resolvePartialId()` which does a `LIKE` prefix match and requires a unique result.
- **JSON fields**: `tags` (string array) and `metadata` (object) are stored as JSON strings in SQLite, serialized on write and parsed on read via `rowToTask()`.
- **Dependency cycle detection**: Uses BFS traversal before inserting a dependency to prevent circular task graphs.
- **Auto-project detection**: CLI detects git root directory and auto-creates/associates a project unless `TODOS_AUTO_PROJECT=false`.
- **Cascade behavior**: Subtask deletion cascades (`ON DELETE CASCADE` via `parent_id`), but project deletion orphans tasks (`ON DELETE SET NULL`).

## Database

bun:sqlite with WAL mode, foreign keys enabled, 5-second busy timeout.

**Schema migrations** (13 total) are version-tracked in a `_migrations` table with forward-only upgrades applied in `getDatabase()`. Migration 5+ uses `ensureTableMigrations()` for backward compatibility with databases that had old SaaS migration IDs.

**Location priority**: `TODOS_DB_PATH` env var → `.todos/todos.db` in cwd → `~/.todos/todos.db` global.

**Core tables**: `projects`, `tasks`, `task_lists`, `agents`, `plans`, `task_dependencies`, `task_comments`, `task_tags`, `sessions`.

## Testing

628 tests across 23 files. Tests use `TODOS_DB_PATH=:memory:` with `resetDatabase()` + `getDatabase()` in `beforeEach` and `closeDatabase()` in `afterEach` for full isolation. CLI integration tests spawn subprocesses with temp DB files created via `mkdtemp`.

Test files: `tasks.test.ts`, `projects.test.ts`, `agents.test.ts`, `task-lists.test.ts`, `plans.test.ts`, `comments.test.ts`, `sessions.test.ts`, `database.test.ts`, `search.test.ts`, `recurrence.test.ts`, `mcp.test.ts`, `errors.test.ts`, `describe.test.ts`, `cli.test.ts`, `completion-guard.test.ts`, `cli-qol.test.ts`, `serve.test.ts`, `agentic.test.ts`.

## MCP Tools (65)

Use `TODOS_PROFILE=minimal|standard|full` to control which tools are registered (reduces cold-start tokens).

**Task CRUD**: `create_task`, `list_tasks`, `get_task`, `update_task`, `delete_task`
**Task Workflow**: `start_task`, `complete_task`, `lock_task`, `unlock_task`, `approve_task`, `fail_task`
**Bulk Ops**: `bulk_update_tasks`, `bulk_create_tasks`
**Agent Coordination**: `get_next_task`, `claim_next_task`, `get_status`, `get_active_work`, `get_tasks_changed_since`, `get_stale_tasks`
**Dependencies**: `add_dependency`, `remove_dependency`
**Comments**: `add_comment`
**Projects**: `create_project`, `list_projects`
**Plans**: `create_plan`, `list_plans`, `get_plan`, `update_plan`, `delete_plan`
**Agents**: `register_agent`, `list_agents`, `get_agent`, `rename_agent`, `delete_agent`, `get_my_tasks`, `get_org_chart`, `set_reports_to`
**Task Lists**: `create_task_list`, `list_task_lists`, `get_task_list`, `update_task_list`, `delete_task_list`
**Search**: `search_tasks` (supports status, priority, date, dependency, assignment filters)
**Analytics**: `get_task_stats`, `get_task_graph`, `clone_task`, `move_task`
**Audit**: `get_task_history`, `get_recent_activity`
**Webhooks**: `create_webhook`, `list_webhooks`, `delete_webhook`
**Templates**: `create_template`, `list_templates`, `create_task_from_template`, `delete_template`
**Sync**: `sync`
**Meta**: `search_tools`, `describe_tools`

Resources: `todos://tasks`, `todos://projects`, `todos://agents`, `todos://task-lists`

## Recurring Tasks

Tasks can have a `recurrence_rule` (e.g. "every day", "every weekday", "every monday", "every 2 weeks"). Completing a recurring task auto-creates the next instance with the correct `due_at`. Pass `skip_recurrence: true` to `complete_task` to stop the chain.

## Webhook Events

`dispatchWebhook` fires on: `task.created`, `task.started`, `task.completed`, `task.failed`, `task.assigned`, `task.status_changed`. HMAC-signed. Register via `create_webhook`.

## Dispatch

Send tasks or task lists to tmux windows — useful for multi-agent workflows where one agent queues work for another.

**Architecture:**
- `src/db/dispatches.ts` — CRUD for dispatch jobs and delivery logs (`dispatches`, `dispatch_logs` tables)
- `src/lib/tmux.ts` — `parseTmuxTarget`, `validateTmuxTarget`, `sendToTmux`, `calculateDelay` (3–5s auto-scaling by message length)
- `src/lib/dispatch-formatter.ts` — `formatDispatchMessage` (task/list → clean numbered text)
- `src/lib/dispatch.ts` — `executeDispatch`, `runDueDispatches`, `dispatchToMultiple`

**CLI:**
```bash
todos dispatch <target> --tasks <id,...>       # Dispatch specific tasks
todos dispatch <target> --list <list-id>       # Dispatch a task list
todos dispatch <target> --tasks <id> --at <iso-datetime>  # Scheduled
todos dispatch <target> --tasks <id> --dry-run # Preview only
todos dispatch <target> --tasks <id> --multiple <t1,t2>  # Fan-out
todos dispatch run [--dry-run]                 # Fire all due dispatches
todos dispatches [--status pending|sent|failed|cancelled] [--limit 20]
todos dispatches --cancel <dispatch-id>
```

**MCP tools:** `dispatch_tasks`, `dispatch_task_list`, `dispatch_to_multiple`, `list_dispatches`, `cancel_dispatch`, `run_due_dispatches`

**Target formats:** `window`, `session:window`, `session:window.pane`

## CLI Agent Commands

```bash
todos next [--agent <id>]        # Best pending task
todos claim <agent>              # Atomic pick + lock + start
todos status [--project <id>]   # Full health snapshot
todos fail <id> [--reason] [--retry]
todos active                     # All in-progress tasks
todos stale [--minutes <n>]     # Stuck tasks
todos done <id> --attach-ids a,b --commit-hash abc --notes "..."
```

## Validation

- **MCP layer**: Zod schemas validate all incoming tool parameters at runtime, with `.describe()` strings generating MCP parameter descriptions
- **Database layer**: SQL CHECK constraints enforce status/priority enums and prevent self-referential dependencies
- **CLI**: Commander.js handles argument parsing; no explicit Zod validation

## Error Classes

Custom errors in `src/types/`: `VersionConflictError`, `TaskNotFoundError`, `ProjectNotFoundError`, `PlanNotFoundError`, `LockError`, `DependencyCycleError`, `AgentNotFoundError`, `TaskListNotFoundError`, `DispatchNotFoundError`. All three surfaces catch and format these consistently.

## TypeScript

Strict mode with `noUncheckedIndexedAccess`, `noUnusedLocals`, `noUnusedParameters`. JSX uses `react-jsx` transform for Ink components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hasna)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hasna)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
