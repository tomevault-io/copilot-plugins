## groundhog-researcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`groundhog-researcher` is an LLM-powered iterative function-optimization framework. A user defines a `Task` (data + context + evaluator), an optimizer iterates with one or more `Strategy` instances, and every `Attempt` is persisted in an immutable tree under `attempts/`.

The package is published to PyPI as `groundhog-researcher` and exposes the `groundhog` / `ghg` CLI for scaffolding tasks.

## Common commands

```bash
# Test suite (~336 tests, fast — runs in under a minute)
pytest

# Iterating on a single area (preferred during work)
pytest tests/test_concepts.py -x          # core primitives — fastest, broadest coverage
pytest tests/test_agent_concepts.py        # agent backends, AgentStrategy, sandbox behavior

# A single test
pytest tests/test_concepts.py -k <name>

# CLI (after installing the package or running from source)
groundhog backends                        # show discovered backends and tier assignments
groundhog init my_task                    # scaffold a task (installs the session skills too)
groundhog prefer <backend>                # preference written to ~/.groundhog/config.json
groundhog attempt <sub>                   # manual attempt lifecycle -- new (--fresh), list,
                                          #   show, commit (gated, --strategy), abort, ...
groundhog eval <path-or-id>               # score a solution dir, .py file, or attempt
groundhog tool list|run                   # run toolkit tools (check-gates is a default)
groundhog skills install [dir]            # copy the packaged session skills into a run dir

# Sandbox probe — runs each agent backend through a 9-op gauntlet, reports filesystem ground truth.
# Use before changes that touch agent sandboxing. Costs API/CLI calls.
uv run tools/probe_agents.py              # all available backends
uv run tools/probe_agents.py codex_cli    # one backend

# Live MNIST smoke (NOT a pytest test — costs money + time, run by user, not automatically)
# Use when changes touch strategies/, optimizers/, agents/, or tools/attempt_log*.py.
cd tests/e2e_mnist_agent && uv run task.py claude 1   # <backend> <iterations>; backends: claude, copilot, codex
cd tests/e2e_mnist_agent && uv run task.py llm 4      # LLM-strategy rotation (Improve/Cross/Fresh) on the cheapest available auth
```

## Design principles (check new code against these)

- **base/ is interfaces only.** No implementation in base/; base classes are the essence of the concept, nothing more.
- **Strategies own the full loop.** Select prior → workspace → generate → evaluate → record. Strategy-specific logic never leaks into the optimizer.
- **Composed method pattern.** Strategy `__call__` reads like a story of named step methods; details live in the steps.
- **Raw results, never scores.** Attempts store metrics dicts; scoring is read-side via per-stage scorers. Never persist a score in the attempt record — the one sanctioned carve-out is the mutable score NOTE (notes.json / git note), a display-only cache that is never read for decisions.
- **Toolkit is capabilities, not tools.** Strategies `hasattr`-check and fall back gracefully; never assume a capability exists.
- **Config is self-documenting.** Every knob is `param(default, "description")`, introspectable via `Config.describe()`.
- **Nothing is discarded.** Every attempt — success or failure — is recorded; failures inform future strategies.
- **Trunks are derived, not stored.** Apply a scorer to the tree to get trunks; never persist trunk membership.

## Releasing

The version lives in ONE place: `src/groundhog/__init__.py` `__version__`
(`pyproject.toml` is hatch-dynamic and reads it). The publish workflow's
version-check job rejects a tag that doesn't equal `__version__`.

Push a tag `v<version>` to trigger the GitHub Actions trusted-publishing workflow that releases to PyPI (it reruns the full test matrix first). Every release is a discrete decision — confirm with the user before commit/push/tag.

Full pre-release checklist: `checklist.md` (repo root) — the consistency manifest covering everything tests can't catch (templates, README, vault alignment, model IDs). Run it before any release commit. CI runs the whole suite via `pytest tests/` — new test files are auto-discovered, no workflow edits needed.

## Architecture

Four layers, each implemented in its own subpackage. `base/` defines interfaces only; everything else is implementations a user can subclass or swap.

```
USER CODE  →  OPTIMIZER  →  STRATEGY  →  AGENT BACKEND  →  AGENT CLI
                                       (or LLM BACKEND for non-agent strategies)
```

### `base/` — interfaces

- **`Task`** = `Data + Context + Evaluator`. The problem definition.
- **`Evaluator.get_stages()`** returns ordered `EvalStage`s (cheap → expensive). Each stage has a `scorer` callable that maps `StageResult.metrics` to a float — **scores are computed, not persisted**, so changing the scorer reinterprets history without re-running anything.
- **`Strategy.__call__(toolkit, config=None)`** is the unit of progress. It selects a prior, creates a `Workspace`, generates+evaluates a candidate, and `ws.commit(result, ...)` writes an immutable `Attempt`.
- **`Toolkit`** is a `SimpleNamespace`-like container the optimizer hands to strategies (holds `task`, `history`, `learnings`, `llm`, `agent`, `agent_tools`, `attempt_logger`, `attempt_log`, `ws` (the attempt pointer), `gates` (GateKit — pure legitimacy facts, utils/gates.py), `finalize` (the standard finish — promote/gates/record/commit/score-note in one call, utils/finalize.py), plus user-added capabilities). `Toolkit.__setattr__` warns when overriding a public attribute; **underscore-prefixed names skip the warning** — use `_foo` for private bookkeeping the optimizer passes through (e.g. `_current_queue_label`).
- **`AttemptHistory`** is the immutable-tree interface (`workspace(parent)`, `get(num)`, iteration, `best(scorer)` — re-scores on demand, never persists scores). `FolderAttemptHistory` is the default implementation; its on-disk convention (`attempts/<num>_<parent>/` directories with `solution.py`, `result.json`, `attemptlog.jsonl`/`.md`, agent raw streams) is implementation detail, not part of the interface contract — other backends (DB, object store, etc.) can implement the same interface differently.
- **`AgentBackend.run(spec: AgentSpec) → AgentResult`** is the contract every CLI agent wrapper implements. `AgentSpec` carries `goal`, `workspace`, `tools` (a list of `AgentTool`s exposed via the HTTP tool server), `allowed_tools` / `denied_tools` (permission rules), `model`, `effort`, `budget_usd`, `session_id`, and `on_event` (callback fired per streamed event — **every backend's subprocess loop must call this**).

### `strategies/` — two families

**LLM strategies** call `toolkit.llm.get(tier)` for a single-turn completion: `Improve`, `FreshApproach`, `CrossPollinate`, `PlanApproaches`, `Analyse`.

**Agent strategies** spawn a CLI agent across multiple phases (explore → submit → evaluate → fix → reflect), all sharing a `session_id`: `AgentStrategy`, `FreshAgentStrategy`, `CrossPollinateAgent`. The phase orchestration, prompts, and base permissions all live in `src/groundhog/strategies/agent.py`. Per-backend event shapes are normalized in `_normalize_agent_event` there — adding a new backend means adding a branch to that function.

### `optimizers/` — one implementation

`SimpleOptimizer` builds the `Toolkit`, loops `n` times, runs strategies in a weighted rotation (`[(strategy, weight), ...]`), seeds with `seed_strategy` if the history is empty, and routes attempt-level emissions through `toolkit.attempt_logger` (typed events → `attemptlog.jsonl`/`.md` + the live console box, which auto-detects TTY vs CI/log mode).

### `agents/` — five CLI wrappers

`claude_code`, `copilot`, `codex_cli`, `gemini_cli`, `opencode`. Each wrapper:

1. Starts an HTTP `ToolServer` exposing `AgentSpec.tools` on a localhost port.
2. Generates per-tool Python wrapper scripts on `PATH` (stdlib `urllib` POST to `localhost:PORT/{tool}`; on Windows each tool also gets `.cmd`, `.ps1`, and bash launchers).
3. Translates `allowed_tools` / `denied_tools` into the CLI's native permission flags (or generated config file).
4. Spawns the CLI as a subprocess in `cwd=spec.workspace`, streams events to `agent_steps.jsonl` + `agent_summary.jsonl`, and **fires `spec.on_event(event)` per event**.

### `backends/` — direct LLM API wrappers

`AnthropicBackend`, `GeminiBackend`, `OpenAICompatibleBackend` (with class methods `.openai`, `.deepseek`, `.groq`, `.cerebras`, `.xai`, `.together`, `.fireworks`, `.ollama`, `.openrouter`), plus CLI-as-backend mirrors (`ClaudeCodeBackend`, `CopilotBackend`, `GeminiCLIBackend`, `OpenCodeBackend`) that drive the same CLIs as `agents/` but for one-shot LLM calls instead of multi-turn agent runs. `discover.discover_backends()` + `auto_registry()` pick what's installed/authenticated and assign tiers (`max`, `high`, `default`, `budget`, `cheap`).

## Sandbox contract for agent strategies

Agents must write only to `attempt/work/`. Reads from `attempt/` (sibling solutions, prior approaches) are allowed. Enforcement varies per backend — `claude_code` and `opencode` are fully enforced via permission rules, `codex_cli` on Windows is advisory-only due to AppContainer ACL limits, `copilot` has known upstream gaps. See `docs/sandboxing.md` for the per-backend matrix and `docs/agent_system.md` for the full phase/tool/permission flow.

`BASE_PERMISSIONS` in `strategies/agent.py` is the source of truth for the read/write/execute rules; per-phase additions/removals are layered on top.

## Repo-specific gotchas

**Cross-platform requirement**: code must work on at least Linux and Windows. Don't reach for OS-specific features (Linux signal handlers, Windows path separators in literals, etc.) without a portable fallback. Use `pathlib.Path`, not string-concat with `/`.

**Windows console gotchas** (when running on Windows): avoid non-ASCII separators like `·` or `→` in stdout — Windows codepage renders them as `�`. Use `|`, `:`, `-`. Box-drawing characters (`╭─╮│╰╯`) work because the terminal is UTF-8.

**npm `.cmd` shims truncate multi-line argv on Windows** at the first `\n` (cmd.exe interprets newlines as command terminators in `%*` expansion). When wrapping a Node CLI, resolve to the `.exe` directly or call `node path/to/bundle.js`. Symptom: model receives only the first line of the prompt. Linux is unaffected — the shim issue is cmd.exe-specific.

**Removed patterns, do not reintroduce**: (a) `self.log.start / inline / tock` as the per-attempt rendering surface (replaced by `AttemptLog` in v0.2.17; two plain-LLM strategies still use `StrategyLog` for console progress lines only); (b) `conversation_log` / per-attempt `conversation.json` (replaced by the typed event stream). Emit per-attempt events with `toolkit.attempt_logger.log(UserEvent/AssistantEvent/ToolCallEvent/...)`; lifecycle via `.attempt_start(ws.path, ...)`, `.attempt_done(...)`, `.attempt_failed(...)`. Attempt cost is **derived from the log** (`logger.total_cost()`), not accumulated on the strategy.

**Claude permission semantics**: `deny > allow`. A narrow allow is shadowed by a broader deny — check both lists when permissions misbehave.

## Style for this repo

- Default to no comments. Add one only when the WHY is non-obvious (hidden invariant, workaround for a specific bug, surprising behavior).
- No backwards-compat shims, no `_` rename of "unused" vars, no `// removed` placeholders.
- Trust internal code; only validate at boundaries (user input, external APIs, subprocess output).
- Match scope: a bug fix shouldn't touch unrelated files.

---
> Source: [frizzerdk/groundhog-researcher](https://github.com/frizzerdk/groundhog-researcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
