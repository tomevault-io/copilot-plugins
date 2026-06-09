## multi-agent-software-team

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install
pip install -r requirements.txt

# Run pipeline (stub mode, no API keys needed)
python cli.py run --issue "fix widget parser race condition" --stub
python cli.py run --issue "..." --stub --coders 4 --baseline

# Run with real LLMs (requires API keys)
python cli.py run --issue "https://github.com/user/repo/issues/42" --repo /path/to/repo

# Batch evaluation
python cli.py eval --issues-file issues.txt --coders 4 --stub --baseline

# Single-agent baseline
python cli.py baseline --issue "..." --stub

# Tests
pytest tests/ -v                                         # all tests
pytest tests/test_task_board.py -v                       # task board (8 tests)
pytest tests/test_roles.py -v                            # roles (9 tests)
pytest tests/test_orchestrator.py -v                     # orchestrator (5 tests)
```

## Architecture

The repo implements a 5-role agent team that converts GitHub issues into mergeable PRs. The pipeline is: **Architect → Coders (parallel) → Merge Coordinator → Reviewer → Tester**, with feedback loops for revision and re-testing.

### Pipeline flow (`src/orchestrator.py:52-277`)

1. **Architect** decomposes the issue into 3–8 subtasks as a DAG (names, files, interfaces, dependencies).
2. **Coders** (up to N, default 4) execute subtasks in parallel — each in an isolated `git worktree` on branch `agent/{coder-name}/{subtask-name}`. They receive repo structure + target file contents as context.
3. **Merge Coordinator** merges all coder branches onto a staging branch. On conflict, it invokes an LLM to resolve merge conflicts.
4. **Reviewer** inspects the combined diff. Can reject with feedback targeting a specific coder (up to 3 revision rounds). Has a **self-review guard**: once an author is reviewed, the same reviewer won't re-approve them.
5. **Tester** runs the test suite. On failure, feeds the log back to coder-A for a fix (up to 2 rounds). Supports Docker sandbox mode.

Key config: `inject_bug_rate` controls false-approve detection — when a known bug is injected and the reviewer approves anyway, `ReviewResult.false_approve` is set.

### Task board (`src/task_board.py`)

A2A protocol via JSONL file. 9 message types (`MsgKind` enum): `PLAN_REQUEST → SUBTASK → DIFF_READY → REVIEW_NEEDED → REVIEW_FEEDBACK → APPROVED → TEST_NEEDED → TEST_PASSED → TEST_FAILED → REPLAN_NEEDED`. Thread-safe via `threading.Lock` on writes. Each message carries token counts for accounting.

### LLM abstraction (`src/llm.py`)

`LLMClient` wraps Anthropic, OpenAI, and Google backends. `LLMFactory` caches clients per provider. `LLMResponse` tracks tokens in/out. `LLMClient.chat()` has `@retry` (exponential backoff, 3 attempts).

### Stub mode

Each role has a `DynamicStub*` subclass and a `create_stub_*()` factory. Stubs produce deterministic, keyword-matched outputs with synthetic token counts (`len(input) // 4`). The orchestrator's `use_stubs=True` toggles all 5 roles to stub at once. Stub reviewers do basic pattern-matching for TODOs, bare excepts, print() calls, etc.

### Budget system (`src/budget.py`)

`BudgetGuard` checks projected token use against per-role and total ceilings before each LLM call. `BudgetExceededError` is raised before spending, preventing costly overruns. The orchestrator catches it and continues with remaining roles or returns a partial report.

### Project config (`src/project_config.py`)

Target repos can have `.team.toml` or `.team.yml` to override role models, token limits, coder parallelism, and test commands per project. Loaded via `TeamRunner._load_project_config()` at startup.

## Code conventions

- `from __future__ import annotations` in every file
- Dataclass-heavy: config, messages, results, reports all use `@dataclass`
- No comments — code is self-documenting through naming
- Public methods have full type annotations; use `|` union syntax (Python 3.11+)
- Tests create real git repos in `tempfile.TemporaryDirectory` — no mocking of git operations

---
> Source: [maquekenzhegua/multi-agent-software-team](https://github.com/maquekenzhegua/multi-agent-software-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
