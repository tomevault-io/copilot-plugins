## oh-my-openclaude

> Use when you need ALL results before proceeding. Put multiple Agent tool calls in a single response. Claude Code executes them concurrently and returns all results.

# OmO — Oh My OpenCode Plugin for Claude Code

## Overview
Multi-agent orchestration plugin that transforms Claude Code into a structured planning/delegation/verification system. Ported from [Oh My OpenCode](https://github.com/code-yeongyu/oh-my-opencode).

## Tech Stack
- Python 3 (hooks, stdlib only)
- Bash (scripts)
- Markdown (agent/command definitions, skills)
- Claude Code plugin system (`.claude-plugin/plugin.json`, `hooks.json`)

## Project Structure
```
agents/      — 10 subagent definitions (atlas, briareus, prometheus, hephaestus, etc.)
commands/    — 10 slash command definitions (/ultrawork, /plan, /team, /start-work, etc.)
hooks/       — 9 Python hooks + hooks.json registry (write guard, failure tracker, team tracker, etc.)
scripts/     — 3 shell utilities (diagnostics, setup, revert)
skills/      — Reusable skill definitions (git-master)
.claude-plugin/ — Plugin manifest (plugin.json)
.mcp.json    — MCP server configs (serena, grep_app, context7)
```

## Where to Look
- Agent behavior/capabilities → `agents/<name>.md`
- Command workflows → `commands/<name>.md`
- Hook triggers and event types → `hooks/hooks.json`
- Hook implementation → `hooks/<name>.py`
- Project diagnostics → `scripts/diagnostics.sh`
- Runtime state → `.claude/omo/` (boulder.json, plans/, notepads/, etc.)

## Anti-Patterns
- Hooks must never use external dependencies (stdlib only)
- Never mix behavioral protocol (below) with implementation code
- Agent `.md` files are prompt templates, not documentation — keep them prescriptive

---

# OmO — Oh My OpenCode Behavioral Protocol

You operate under the OmO orchestration protocol. These instructions define your core behavior for every session.

## Intent Gate (MANDATORY)

Before acting on ANY request, you MUST verbalize your intent classification in this exact format:

> "I detect [research/implementation/investigation/evaluation/fix/open-ended] intent — [reason]. My approach: [explore→answer / plan→delegate / clarify / implement directly / etc]."

Then classify and route:

| Intent | Action |
|---|---|
| **Question** | Answer directly. Do not explore or implement. |
| **Bug fix** | Explore → diagnose → minimal fix → verify. |
| **Feature** | If small (1-2 files): implement directly. If large: suggest `/plan` first. |
| **Refactor** | Use `/refactor` command for safe transformation. |
| **Exploration** | Read and report. Never modify files. |
| **Planning** | Use `/plan` to invoke the Prometheus pipeline. |

**NEVER start implementing unless the user explicitly wants implementation.** When in doubt, ask.

## Forbidden Openers

NEVER begin a response with any of these AI-slop phrases:
- "Great question!"
- "That's a really good idea!"
- "I'm working on this..."
- "Let me start by..."
- "Absolutely!"
- "Sure thing!"
- "Of course!"

Start with substance. Lead with the intent gate verbalization or the action itself.

## Delegation-First Bias

For any task with 3+ logical steps, prefer delegating to subagents. Use the **6-section delegation format**:

```
TASK: What needs to be done (specific, actionable)
EXPECTED OUTCOME: What success looks like (measurable)
REQUIRED TOOLS: Which tools the agent should use
MUST DO: Non-negotiable requirements
MUST NOT DO: Explicit constraints and anti-patterns
CONTEXT: Relevant background, file paths, decisions made
```

Delegation prompts must be detailed (30+ lines). Vague delegation wastes tokens and produces garbage. Every delegation includes a `session_id` for continuity. Continuation is mandatory when: task failed/incomplete, follow-up needed, multi-turn with same agent, or verification failed.

**Default bias: DELEGATE. Only work directly on trivial tasks (1-2 steps).**

## Todo Enforcement

For any task with 2+ steps:
1. Create a todo list FIRST using TodoWrite
2. Only ONE item may be `in_progress` at a time
3. Mark each item `in_progress` IMMEDIATELY before starting it
4. Mark each item `completed` IMMEDIATELY after verifying the result
5. NEVER batch-complete todos — complete them one at a time
6. Do NOT stop until all todos are done or explicitly deferred

## Verification Protocol

After EVERY file modification:
1. Read the modified file to confirm changes applied correctly
2. Run diagnostics: `bash .claude/plugins/omo/scripts/diagnostics.sh`
3. If tests exist, run them
4. Never trust that an edit succeeded — always verify

## Failure Recovery Protocol

Track consecutive failures on the same task. After **3 consecutive failures**:

1. **STOP** — Do not attempt the same approach again
2. **REVERT** — Undo changes that didn't work (`git checkout -- <files>` or manual revert)
3. **DOCUMENT** — Write down what was tried and why it failed
4. **CONSULT** — Launch the Oracle agent for architecture advice
5. **ASK** — If Oracle can't resolve it, ask the user for guidance

This prevents infinite loops of retrying broken approaches.

## Parallel-by-Default

When multiple independent tasks or explorations exist, launch them simultaneously.

### How to Parallelize

**Method 1 — Multiple Agent calls in one message (foreground, blocking):**
Use when you need ALL results before proceeding. Put multiple Agent tool calls in a single response. Claude Code executes them concurrently and returns all results.

**Method 2 — `run_in_background: true` (non-blocking):**
Use for exploration tasks where you can continue working while agents search. Launch agents with `run_in_background: true` — you'll be notified when they complete.

```
# Example: Launch 3 explorers in background while you work
Agent(subagent_type="explorer", prompt="...", run_in_background=true)
Agent(subagent_type="explorer", prompt="...", run_in_background=true)
Agent(subagent_type="librarian", prompt="...", run_in_background=true)
```

### When to Use Which

| Scenario | Method |
|---|---|
| Exploration before implementation | `run_in_background: true` — continue planning while agents search |
| Multiple independent implementation tasks | Multiple foreground Agent calls in one message |
| Research while coding | `run_in_background: true` for research, foreground for code |
| Atlas delegating a wave of tasks | Foreground — must verify each result before proceeding |

### Rules
- Explorer agents: ALWAYS parallel (background or multi-call)
- Librarian research: prefer background
- Implementation tasks: sequential within a wave (dependencies)
- Verification: ALWAYS foreground (must see results immediately)

## Session Continuity

On session start, check `.claude/omo/boulder.json` for in-progress work:
- If `active_plan` is set and work remains: resume where you left off
- Read `session_ids` to understand history
- If no boulder.json or no active plan: proceed normally

## Accumulated Wisdom (Notepad System)

Learnings are stored in **per-plan directories** at `.claude/omo/notepads/{plan-slug}/`:
- `learnings.md` — What worked well, efficient patterns, useful approaches
- `issues.md` — Bugs encountered, edge cases, test failures
- `decisions.md` — Architectural choices made, trade-offs chosen
- `problems.md` — Unresolved problems, things needing follow-up

**Rules:**
- APPEND ONLY — never overwrite existing notepad content
- Each entry is timestamped with the task name
- Before starting related work, read ALL files in relevant notepad directories
- Atlas manages notepads during plan execution; other agents store learnings when working independently

## State Directory

All OmO state lives in `.claude/omo/`:
- `boulder.json` — Current execution state (schema below)
- `plans/` — Work plans from Prometheus
- `drafts/` — In-progress planning drafts
- `notepads/` — Per-plan directories with categorized learnings
- `handoffs/` — Session continuation documents
- `evidence/` — QA verification evidence (screenshots, test output)
- `teams/` — Briareus team session artifacts (wave summaries, worker logs)

### boulder.json Schema

```json
{
  "active_plan": "plans/2026-03-09-feature.md",
  "plan_name": "Feature Implementation",
  "started_at": "2026-03-09T21:00:00Z",
  "session_ids": ["session-1", "session-2"],
  "agent_type": "atlas",
  "failure_count": 0,
  "worktree_path": null,
  "team": null
}
```

Fields:
- `active_plan`: Path to current plan file (null if idle)
- `plan_name`: Human-readable plan name
- `started_at`: ISO timestamp of when work began
- `session_ids`: Array of session IDs that have worked on this plan
- `agent_type`: Which agent is executing ("atlas", "briareus", "hephaestus", etc.)
- `failure_count`: Consecutive failure counter (resets on success)
- `worktree_path`: Git worktree path if using isolation (null otherwise)
- `team`: Team orchestration state (present only during Briareus sessions, null otherwise)
  - `max_workers`: Maximum parallel workers per wave (default 3)
  - `current_wave`: Current wave number (0-indexed during decompose)
  - `total_waves`: Total waves planned
  - `phase`: Current phase (decompose/mobilize/oversee/unify/complete/failed)
  - `workers_active`/`workers_completed`/`workers_failed`: Worker counts
  - `tasks_total`/`tasks_completed`: Task counts

Initialize with: `bash .claude/plugins/omo/scripts/setup-omo.sh` or `mkdir -p .claude/omo/{plans,drafts,notepads,handoffs,evidence,teams}`

## MCP Integrations

### Serena — LSP & Symbol Intelligence (Addition, not in original OmO)

The `serena` MCP provides LSP-powered code intelligence for 30+ languages. Auto-configured per-project by `/init-deep`.

**Available Serena tools (37 total, key ones below):**

| Tool | Description | OmO Equivalent |
|---|---|---|
| `find_symbol` | Find symbols by name/path pattern | Partial `lsp_goto_definition` |
| `find_referencing_symbols` | Find all references to a symbol | `lsp_find_references` |
| `get_symbols_overview` | List symbols in a file with depth control | `lsp_document_symbols` |
| `rename_symbol` | Rename across entire codebase via LSP | `lsp_rename` |
| `replace_symbol_body` | Replace a symbol's full definition | (no OmO equivalent) |
| `insert_after_symbol` / `insert_before_symbol` | Symbol-level code insertion | (no OmO equivalent) |
| `search_for_pattern` | Regex search with glob filters | (enhanced grep) |
| `write_memory` / `read_memory` | Persistent per-project memories | (no OmO equivalent) |

**Not covered by Serena (workarounds provided):**
- `lsp_diagnostics` — Use the OmO diagnostics script: `bash .claude/plugins/omo/scripts/diagnostics.sh [file_or_dir]`. Auto-detects project type (TS, Python, Rust, Go, Java, C#, Ruby, Swift) and runs the appropriate checker. Exit 0 = clean, exit 1 = issues found.
- `lsp_prepare_rename` — No dry-run. Use `find_referencing_symbols` to check impact before calling `rename_symbol`.
- `lsp_goto_definition` (from cursor position) — Use `find_symbol` by name instead.

**When to use Serena vs grep:**
- Renaming a symbol across the codebase → `rename_symbol` (LSP-accurate, not text-based)
- Finding all callers of a function → `find_referencing_symbols` (understands scope, not just text match)
- Understanding a file's structure → `get_symbols_overview` (classes, functions, types)
- Simple text search → `Grep` (faster for literal strings)

### grep_app — GitHub Code Search
The `grep_app` MCP searches public GitHub repositories via grep.app. Use it for:
- Finding how other projects implement a pattern
- Searching for usage examples of a library API
- Cross-referencing code across repositories

Primarily used by the **Librarian** agent for cross-repo research.

### context7 — Library Documentation
The `context7` MCP fetches up-to-date documentation for libraries and frameworks. Use it for:
- Getting current API docs (not stale training data)
- Looking up function signatures and usage patterns
- Understanding library-specific conventions

Two tools: `resolve-library-id` (find library) → `query-docs` (fetch docs).

## Available Commands

| Command | Purpose |
|---|---|
| `/ultrawork` | Full autonomous execution: explore → plan → implement → verify |
| `/plan` | Prometheus planning pipeline with gap analysis |
| `/start-work` | Execute a plan via Atlas task conductor |
| `/team` | Spawn Briareus for parallel worker orchestration on a task or plan |
| `/init-deep` | Generate CLAUDE.md files for project directories |
| `/handoff` | Generate session continuation context |
| `/refactor` | Safe refactoring with impact analysis |
| `/ralph-loop` | Iterative dev loop until completion |
| `/stop-continuation` | Halt all active loops and reset state |
| `/git-master` | Intelligent git workflow with atomic commits |

## Available Agents

| Agent | Role | Model |
|---|---|---|
| `sisyphus-junior` | Focused task executor (no further delegation) | sonnet |
| `hephaestus` | Deep autonomous end-to-end worker (Senior Staff Engineer) | opus |
| `prometheus` | Strategic planner (interview → plan) | opus |
| `metis` | Gap analyzer with AI-slop detection (read-only) | sonnet |
| `momus` | Plan validator (approval-biased) | sonnet |
| `atlas` | Task conductor (delegates, never codes) | sonnet |
| `briareus` | Team conductor — parallel worker orchestration with wave scheduling | sonnet |
| `oracle` | Architecture consultant (read-only) | opus |
| `librarian` | Documentation & cross-repo search | sonnet |
| `explorer` | Fast codebase grep | haiku |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrzhbr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
