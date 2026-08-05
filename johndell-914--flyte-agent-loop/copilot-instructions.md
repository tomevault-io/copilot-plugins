## flyte-agent-loop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A "loop-engineering" system: three scheduled Flyte 2 agent pipelines (`flyte.ai.agents.Agent`) that cooperate over shared durable memory to take GitHub issues to reviewed PRs autonomously, then grade themselves. Full design rationale lives in `docs/architecture.md` — read it before changing pipeline logic.

## Commands

```bash
# Environment (uv-managed)
uv venv --python 3.13 && source .venv/bin/activate   # Windows: .venv\Scripts\activate
uv pip install -e ".[dev]"

# Tests — default collection is scoped to tests/unit via pyproject testpaths
pytest -q                                  # hermetic unit suite (no cluster/network/LLM)
pytest tests/unit/test_dibs.py -q          # one file
pytest tests/unit/test_dibs.py::test_name  # one test

# Lint (what CI runs)
ruff check src tests

# Integration tests — manual, hits a live Union tenant, skipped unless opted in
RUN_INTEGRATION=1 pytest tests/integration -m integration -s

# Run one pipeline locally against a real repo (needs FLYTE_AGENT_REPO,
# GITHUB_TOKEN, ANTHROPIC_API_KEY env vars — see .env.example)
python examples/run_local.py {builder|reviewer|distiller}

# Deploy to a Flyte cluster (uses ./config.yaml or ~/.flyte/config.yaml)
python -m flyte_agent_loop.deploy              # deploy + activate cron triggers
python -m flyte_agent_loop.deploy --dryrun     # plan only
python -m flyte_agent_loop.deploy --run builder  # trigger one run ad hoc
```

CI (`.github/workflows/unit-tests.yml`) runs `ruff check src tests` + `pytest tests/unit -q` on Python 3.11–3.13.

## Architecture

Three pipelines, each a single `@env.task(report=True, triggers=[...])` on one shared `flyte.TaskEnvironment` (`environments.py`):

- **`builder_agent.py`** (cron */5) — claims an open issue, implements it (or decomposes a spec issue into sub-issues), verifies, opens a PR / files issues.
- **`reviewer_agent.py`** (cron */5) — claims an agent-authored PR, reads review comments, makes verified fixes, pushes them, releases the claim.
- **`distiller_agent.py`** (cron */10) — folds builder/reviewer run history into a compact "lessons" memory via a distiller Agent, publishes metrics/memory/trace report tabs.

Core patterns that span multiple files:

- **Propose → verify → durable write.** Builder/reviewer agents never write to GitHub directly. They stage a proposal; a stricter verifier sub-agent (`agents.py`) must return `{"verified": true}` before the pipeline performs the durable write (`open_pr_with_changes`, `push_changes_to_pr`). On failure, verifier feedback is fed back for up to `FLYTE_AGENT_MAX_TRIES` attempts.
- **Two kinds of agent tools.** `tools.py` tools are `@env.task` — each call dispatches a tracked, durable Flyte sub-action. `staging.py` tools (`stage_file`, `submit_implementation`, …) are *plain closure tools* invoked in-process, accumulating into a `ChangeStage` the pipeline owns. Agents stage one file per call (with `parallel_tool_calls=False`) specifically to avoid max-output-token truncation of a single giant JSON blob — don't refactor back to whole-plan-in-one-message.
- **Dibs (cooperative locking), `dibs.py`.** A pure state machine over invisible HTML-comment markers on issues/PRs. Ownership is per-*run* (not per-agent); races resolve via claim-then-read-back FCFS, plus an authoritative pre-open PR guard in the builder. Pure by design (comments + explicit `now`) so it's fully unit-testable — keep it network-free.
- **Shared memory, `memory_context.py`.** Two keyed `MemoryStore`s with deliberately disjoint writer sets: `<key>-runs` (written by builder/reviewer at unique paths, read by distiller) and `<key>-context` (written only by distiller, read by builder/reviewer). The split exists because `MemoryStore.save()` re-uploads the whole local snapshot — a single shared store would let one pipeline revert another's concurrent write.
- **Config flow.** `config.py` reads everything from env vars (`FLYTE_AGENT_*`); `environments.py` captures those set at deploy time into the task environment's `env_vars` and injects the `github-token` / `anthropic-api-key` cluster secrets as `GITHUB_TOKEN` / `ANTHROPIC_API_KEY`. See `.env.example` for the full list.
- **Error recovery.** Each pipeline wraps its flow in a top-level `try/except`: on any error it releases the dibs claim, records an `error` RunRecord, and surfaces the error in the report rather than crashing. Tasks run with `retries=0` — degradation is deliberate. Work is chunked into named stages via `flyte.group(...)` for the Flyte UI timeline.

## Testing strategy

Non-trivial logic is isolated into pure, hermetic modules so `tests/unit` needs no cluster, network, or LLM key: `dibs.py` (claim state machine), `evals.py` (metrics/compaction), `agents.py` (plan/verdict parsers), and `github_client.py` (tested against an in-memory `httpx.MockTransport` via respx). Pipeline modules and `tools.py` are thin orchestration over these units — when adding logic, put it in a pure module and test it there.

## Conventions

- The ruff rule set is pinned to `E4, E7, E9, F` on purpose — broad `except Exception` is an intentional pattern for graceful pipeline recovery. Don't enable stricter rule groups or "fix" those handlers. Line length is 110.
- Python ≥3.11; `pytest-asyncio` runs in `asyncio_mode = "auto"` (async tests need no decorator).
- The slide deck (`docs/index.html`) is a single self-contained HTML file with no build step, published to GitHub Pages by `.github/workflows/pages.yml` on pushes to `main` touching `docs/`.

---
> Source: [johndell-914/flyte-agent-loop](https://github.com/johndell-914/flyte-agent-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
