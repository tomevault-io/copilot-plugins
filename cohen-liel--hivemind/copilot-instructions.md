## hivemind

> Hivemind is a multi-agent software engineering orchestrator. It takes a user prompt, decomposes it into a DAG of tasks, assigns specialist agents, executes them with dependency awareness, and delivers tested/reviewed code.

# CLAUDE.md — Hivemind Project Guide

## What is this project?

Hivemind is a multi-agent software engineering orchestrator. It takes a user prompt, decomposes it into a DAG of tasks, assigns specialist agents, executes them with dependency awareness, and delivers tested/reviewed code.

## Architecture Overview

```
User message
  → Triage (_triage_is_simple)         # Simple? Skip PM+Architect
  → Architect Agent (architect_agent)  # Codebase review → ArchitectureBrief
  → PM Agent (pm_agent)                # Decompose → TaskGraph (DAG)
  → LangGraph DAG Executor             # select_batch → execute_batch → post_batch → loop
  → Review (read-only critique)        # ACC-Collab pattern + lint/format + test safety net
  → Memory Agent                       # Update project memory with lessons learned
```

### Key Design Decisions

- **One DAG per project, always.** New messages inject tasks into the live DAG (add or cancel). Never spawn parallel DAGs on the same project. Deferred messages are buffered during PM/Architect phase and drained when the graph is ready.
- **Read-only reviewer.** The reviewer critiques code but never modifies it. Automated lint/format runs separately. If lint/format breaks tests, changes are reverted to pre-review HEAD.
- **Adaptive triage.** Simple tasks (short prompt, no complex keywords) skip PM + Architect and go straight to a single fullstack agent. Complex tasks get the full pipeline.
- **Writer/Reader separation.** Writer agents (modify files) run sequentially under `asyncio.Lock`. Reader agents (analysis, research) run in parallel.
- **Typed contracts.** `TaskInput → TaskOutput` with structured artifacts, not free-form text.

## Key Files

| File | Purpose |
|---|---|
| `orchestrator.py` | Central coordinator — session lifecycle, triage, DAG dispatch, message injection, event emission |
| `dag_executor_langgraph.py` | LangGraph StateGraph: `select_batch → execute_batch → post_batch → review_code`. SQLite checkpointing. Self-healing. |
| `pm_agent.py` | PM Agent — decomposes user requests into `TaskGraph` DAGs. Task count scales with complexity (no forced minimums). |
| `architect_agent.py` | Architect Agent — pre-planning codebase review, produces `ArchitectureBrief` |
| `contracts.py` | `TaskInput`, `TaskOutput`, `TaskGraph`, `TaskStatus` — typed contracts for all agent communication |
| `config.py` | All configuration constants (`DAG_MAX_CONCURRENT_NODES`, agent registry, timeouts, budgets) |
| `src/workers/task_queue.py` | `ProjectTaskQueue` with per-project `asyncio.Lock` for writer serialization |
| `memory_agent.py` | Post-execution memory updates + lessons-learned injection |
| `blackboard.py` | Shared state for complexity classification, inter-agent notes |
| `debate_engine.py` | Structured debate for critical tasks (currently proactive, planned: reactive on failure) |
| `reflexion.py` | Self-reflection on failed tasks before retry |
| `orch_watchdog.py` | Agent silence detection (5 stuck signals) + active escalation |

## Conventions

- **Python 3.11+**, async-first (`asyncio`). No sync blocking in the event loop.
- **Linting:** `ruff check` + `ruff format`. Run before committing.
- **Commits:** Conventional commits (`feat:`, `fix:`, `chore:`). One logical change per commit.
- **No forced minimums:** PM agent scales task count to request complexity. A 1-task plan is valid for simple requests.
- **No parallel DAGs on same project.** This is a hard invariant. Messages buffer, inject, or wait — never spawn a second DAG.

## Testing

```bash
python3 -m pytest tests/ -v          # Full suite
python3 -m pytest tests/ -k "test_name"  # Specific test
python3 -c "import py_compile; py_compile.compile('file.py', doraise=True)"  # Quick syntax check
```

## Common Patterns

### Adding a new agent role
1. Add the role to `AgentRole` enum in `contracts.py`
2. Add agent config in `config.py` `AGENT_REGISTRY`
3. Add system prompt in `prompts.py`
4. The DAG executor will automatically dispatch to it based on `TaskInput.role`

### Modifying the DAG execution loop
The LangGraph graph is built in `build_dag_graph()` at the bottom of `dag_executor_langgraph.py`:
```
START → select_batch → execute_batch → post_batch → should_continue → (loop or review_code or END)
```
Each node is an async function that takes `DAGState` and returns a partial state update dict.

### Injecting tasks mid-execution
`orchestrator.py:_inject_user_tasks()` calls PM to decompose the new message, then uses `graph.add_task()` / `graph.remove_task()` to modify the live TaskGraph. The executor's `select_batch` will discover new tasks via `ready_tasks()` in the next round.

---
> Source: [cohen-liel/hivemind](https://github.com/cohen-liel/hivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
