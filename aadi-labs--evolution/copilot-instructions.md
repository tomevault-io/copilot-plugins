## evolution

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Evolution is a multi-agent code evolution platform. It spawns autonomous AI agents (Claude Code, Codex, OpenCode) into isolated workspaces, each working on the same codebase + grading function. Agents research, implement, evaluate, and share knowledge to push a score as high (or low) as possible.

## Commands

```bash
# Install dependencies
uv sync --dev

# Run all tests (328 tests, pure unit tests, no external services needed)
uv run pytest tests/ -q

# Run a single test file
uv run pytest tests/test_config.py -q

# Run a single test
uv run pytest tests/test_config.py::test_heartbeat_defaults -q

# Lint
uv run ruff check evolution/

# Lint with auto-fix
uv run ruff check evolution/ --fix

# Run the CLI (after uv sync)
uv run evolution --help
```

## Architecture

### Communication Model

All shared-state writes are serialized through the **Manager** via a Unix domain socket (`.evolution/manager.sock`). Agents talk to the manager by sending newline-delimited JSON over the socket. The CLI commands (`evolution eval`, `evolution note add`, etc.) are thin clients that build a JSON request and call `send_request()`. This is the single most important architectural invariant — zero race conditions by construction.

### Package Layout

- **`evolution/manager/`** — Core orchestrator. `Manager` owns the lifecycle: spawns agents, runs grading, tracks scores, handles heartbeats, detects stagnation. `ManagerServer` is the Unix socket server. `config.py` has all Pydantic models for `evolution.yaml` parsing.
- **`evolution/adapters/`** — Runtime adapters. `AgentAdapter` (base class) defines three methods: `provision()`, `write_instructions()`, `spawn()`, plus `clean_env()` which strips `VIRTUAL_ENV`/`PYTHONPATH`/`PYTHONHOME` from subprocess environments. Each runtime (Claude Code, Codex, OpenCode) is one subclass. Adding a runtime = one new file. Adapters derive permission flags from `AgentConfig.runtime_options` (e.g., `permission_mode: dangerously-skip-permissions`).
- **`evolution/hub/`** — Shared knowledge layer. Four hub classes (`AttemptsHub`, `NotesHub`, `SkillsHub`, `HypothesisHub`) each manage a directory of markdown files with YAML frontmatter. This is the file-as-truth pattern — no database.
- **`evolution/grader/`** — Grading strategies. `GradeResult` (in `protocol.py`) is the common return type. `script.py` runs a Python script (with `clean_env()` to prevent venv leakage; timeout configurable via `task.grader.timeout`), `llm.py` sends diffs to an LLM, `hybrid.py` combines both. `ranking.py` has multi-metric comparison (weighted sum, Pareto dominance, min-rank, all-must-improve). `jury.py` aggregates multiple grader opinions.
- **`evolution/workspace/`** — Workspace isolation. `WorkspaceManager` auto-detects reflink support (APFS/btrfs) for CoW clones, falling back to git worktrees with auto-discovered symlinks for gitignored dirs. `discovery.py` finds untracked directories to symlink. `reflink.py` handles CoW copy + disk cache warming.
- **`evolution/cli/`** — CLI entry point. `main.py` builds the argparse tree and dispatches. `init.py` bootstraps `evolution.yaml` + grader. `run.py` starts the manager. `report.py` generates session reports.
- **`evolution/superagent/`** — Optional meta-agent that monitors and steers the session.

### Key Data Flow

1. Agent calls `evolution eval -m "description"` from its worktree
2. CLI detects agent name from cwd path (`.evolution/worktrees/<name>/`)
3. CLI sends JSON request to manager socket
4. Manager queues eval in `EvalQueue` (thread-safe, supports rate limiting + round-robin fairness)
5. Manager runs grader in agent's worktree, gets `GradeResult`
6. Manager records attempt via `AttemptsHub` (writes markdown file to `.evolution/shared/attempts/`)
7. Manager delivers result to agent's inbox (`.evolution/inbox/` in worktree)
8. Heartbeat trackers (`MultiHeartbeatTracker`) fire at configured frequencies, triggering reflection/consolidation/convergence messages

### Workspace Layout at Runtime

```
.evolution/
├── manager.sock          # Unix domain socket
├── shared/               # Symlinked into every worktree
│   ├── attempts/         # Markdown files: 001-agent-1-score-0.8500.md
│   ├── notes/            # Agent observations, findings, warnings
│   ├── skills/           # Reusable techniques published by agents
│   └── hypotheses/       # Structured predictions with evidence
└── worktrees/
    ├── agent-1/          # Full repo clone (reflink or git worktree)
    │   └── .evolution/
    │       ├── inbox/    # Messages delivered by manager
    │       └── shared -> ../../shared/
    └── agent-2/
```

### Configuration

Everything is in `evolution.yaml`, parsed into `EvolutionConfig` (Pydantic). The config hierarchy: `SessionConfig` > `TaskConfig` (with `StopConfig`, `EvalQueueConfig`, `PhaseConfig`, `MilestoneConfig`) > `RoleConfig` (with heartbeat) > `AgentConfig` (with `runtime_options: dict[str, Any]` for per-runtime config like `permission_mode`) > `SuperagentConfig`.

### Testing Patterns

Tests are pure unit tests using `tmp_path`, `tempfile`, and direct class instantiation. No mocking frameworks — tests construct real hub/config/grader objects against temp directories. The test files mirror the package structure (e.g., `test_config.py` tests `manager/config.py`, `test_workspace.py` tests `workspace/setup.py`).

---
> Source: [aadi-labs/evolution](https://github.com/aadi-labs/evolution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
