## agent-comms

> <!-- Generated: 2026-03-02 | Updated: 2026-03-02 -->

<!-- Generated: 2026-03-02 | Updated: 2026-03-02 -->

# agent-comms — HIVE Fleet Protocol

## Purpose

Universal inter-agent communication bus for CNC-1. File-based JSONL. Zero
dependencies for the core bus. Multiple AI terminals (Claude Code, Gemini CLI,
OpenClaw) coordinate through append-only JSONL channels. Any process that can
append to a file can participate.

The Python `hive` package adds structured coordination on top: cell schema
validation, task lifecycle enforcement (A2A-aligned 7 states), lease-based
claim locking, DAG dependency tracking, stall detection, reputation scoring,
and a FastAPI dashboard. The `comms.sh` CLI wraps the raw bus for shell
convenience.

## Key Files

| File | Description |
|------|-------------|
| `CLAUDE.md` | Claude Code guidance: commands, architecture, concurrency invariants |
| `comms.sh` | Primary CLI entry point — source this, do not execute directly |
| `agent-runner.sh` | Persistent dispatch loop for non-interactive agents; validates cells before posting |
| `PROTOCOL.md` | Canonical A2A-aligned task lifecycle schema and cell format specification |
| `FLEET-OPS.md` | Post-mortem from 2026-03-02 first live run — 7 mistakes and their fixes |
| `ROLE-PROMPTS.md` | Role-scoped agent prompts (Claude/architect, Gemini/researcher, Codex/deployer) |
| `standards.md` | Fleet operating standards — clock-in/out protocol, mandatory reads on session start |
| `manifest.json` | Machine-readable protocol definition: identity format, channel registry, agent roster |
| `codex-wrap.py` | Wrapper for codex CLI: strips ANSI/progress noise, extracts test counts |
| `pyproject.toml` | Python package config; pytest timeout 30s; mypy strict + ruff config |
| `org.json` | Organizational config (agent and channel ownership metadata) |
| `README.md` | Quick-start guide: commands, channel list, handoff protocol, raw usage |
| `RESEARCH.md` | Research notes on paradigm-shifting multi-agent tools compiled 2026-03-02 |
| `.mcp.json` | MCP server configuration for this project |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `hive/` | HIVE Python protocol library (see `hive/AGENTS.md`) |
| `tests/` | Full pytest suite (see `tests/AGENTS.md`) |
| `channels/` | JSONL channel files — shared message bus (see `channels/AGENTS.md`) |
| `dashboard/` | Factory floor web UI + FastAPI server (see `dashboard/AGENTS.md`) |
| `docs/` | Design documents and implementation plans (see `docs/AGENTS.md`) |

## For AI Agents

### Working In This Directory

- NEVER hardcode credentials
- CHANNELS_DIR = `C:/Users/Brady.EAGLE/.ai/channels` (canonical — NOT `channels/` in this repo)
- Set `COMMS_AGENT` before any comms command: `export COMMS_AGENT="agent/role"`
- Run tests: `cd C:/tools/agent-comms && python -m pytest tests/ --timeout=30 -q`
- Quality gates (all enforced in CI, all must pass before pushing):
  `python -m pytest tests/ -q` && `mypy` && `ruff check .`
- `comms.sh` is the CLI entry point — source it, don't execute directly

### Task Protocol (A2A-Aligned — 7 States)

```
submitted -> working -> blocked -> complete -> failed -> canceled -> verified
```

See `PROTOCOL.md` for full schema. Never post empty cells (msg < 20 chars = violation).

### Never-Again Rules (from FLEET-OPS.md)

- Never use commands that aren't in `comms.sh` or this document — Gemini hallucinated `comms join`, `comms broadcast`, and `/hive-clock-in`; none exist
- Never give an agent a single-task prompt — task-scoped prompts cause agents to go idle after one task; always use role-scoped prompts with a "never stop" directive
- Never post a cell with msg shorter than 20 characters — empty status cells (e.g., just "TASK-3" or "deployer") are protocol violations; `agent-runner.sh` rejects them
- Never start a task before checking its `depends_on` — Codex claimed TASK-3 before TASK-2 was even posted; always verify dependency state is COMPLETE first
- Never write to any channel path other than `C:/Users/Brady.EAGLE/.ai/channels` — the repo's `channels/` directory is not the live bus; split writes corrupt the fleet's shared state
- Never post a result more than once for the same task_id — Gemini posted TASK-2 results twice with different content; one result per task, tracked in agent-runner state file
- Never run comms commands with `COMMS_AGENT` unset or set to "unknown" — agent identity must be set in format `name/role` before any comms operation; `agent-runner.sh` exits immediately otherwise

## Dependencies

### Internal

- `comms.sh` sources from this directory
- `hive/` Python package installed via `pyproject.toml`

### External

- Python 3.11+ (stdlib only for `hive` core)
- `fastapi` + `uvicorn` (dashboard only)
- `pytest` + `pytest-timeout` (tests)

<!-- MANUAL: -->

---
> Source: [BFlinkDesign/agent-comms](https://github.com/BFlinkDesign/agent-comms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
