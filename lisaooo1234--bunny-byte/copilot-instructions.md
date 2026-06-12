## bunny-byte

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development commands

Install the project for development. The README documents pip editable install with the dev extra; the `pyproject.toml` also declares dev tools in `[dependency-groups]`, so `uv sync --dev` is the uv-native setup path.

```bash
pip install -e ".[dev]"
# or
uv sync --dev
```

Run the full test suite with:

```bash
pytest tests/ -q
```

Run a single test file or single test with standard pytest selection:

```bash
pytest tests/test_runtime.py -q
pytest tests/test_runtime.py::test_engine_drives_real_session_and_persists_event_timeline -q
```

Run the live provider smoke test only when provider credentials are configured:

```bash
BUNNYBYTE_LIVE_SMOKE=1 pytest tests/test_release_smoke.py -q
```

Run Ruff linting with:

```bash
ruff check .
```

Start the application from a development checkout with:

```bash
uv run bunnybyte
```

Installed console entry points are `bunnybyte`, `bunnybyte-tui`, `bb`, and `bbtui`. Useful runtime invocations include:

```bash
bunnybyte
bunnybyte --repl
bunnybyte "找出测试失败的根因"
bunnybyte --resume latest
bunnybyte --cwd /path/to/repo
bunnybyte --provider deepseek --approval ask --max-steps 80
```

## Project architecture

Bunny Byte is a local terminal coding agent. It runs in a repository workspace, builds prompts from workspace context, memory, skills, and session history, calls a configured model provider, executes model-requested tools through a guarded registry, and persists sessions, events, traces, reports, and memory under `.bunnybyte/`.

The CLI entry point is `bunnybyte/cli.py`. It parses startup arguments, resolves provider configuration, loads project environment values, builds a `WorkspaceContext`, opens or resumes a `SessionStore`, resolves sandbox settings, and constructs the `BunnyByte` runtime. The TUI in `bunnybyte/tui/` is a presentation layer around the same runtime: `BunnyByteTuiApp` drives turns through `Engine.run_turn()` and handles Textual UI concerns such as chat rendering, slash completion, approval prompts, and ask-user prompts.

Provider configuration lives in `bunnybyte/config/`. Configuration precedence is CLI arguments, then environment variables, then project `.bunnybyte.toml`, then global `~/.config/bunnybyte/config.toml`, then code defaults. Provider profiles separate the profile name from the request protocol: `openai` uses the OpenAI-compatible Responses API path, while `anthropic` covers Anthropic-compatible Messages-style endpoints, including the default DeepSeek profile. The concrete HTTP adapters are in `bunnybyte/providers/clients.py`, while `bunnybyte/providers/base.py` normalizes provider outputs to `ModelResult`.

The central runtime object is `BunnyByte` in `bunnybyte/core/runtime.py`. It owns model client state, workspace identity, session persistence, event bus, run store, memory, skills, tool registry, permission checker, plan mode, worker manager, context manager, and checkpoints. The runtime intentionally delegates turn control to `bunnybyte/core/engine.py`, prompt assembly to `ContextManager`, tool execution guardrails to `tool_executor.py`, and smaller state/persistence concerns to focused modules such as `task_state.py`, `run_store.py`, `session_store.py`, `session_events.py`, `todo_ledger.py`, `permissions.py`, `tool_policy.py`, and `plan_mode.py`.

A single user turn is controlled by `Engine.run_turn()`. It creates a `TaskState` and run directory, records the user message, asks `ContextManager` to build a budgeted prompt, streams model deltas through `model_stream.complete_model_with_deltas()`, parses model output into final/retry/tool forms, executes requested tools, and persists session events plus run evidence. Successful turns promote memory, run memory maintenance, create a checkpoint, write task state, and write a run report. Provider errors are captured as failed runs with provider metadata rather than hidden from the user.

Prompt construction is handled by `bunnybyte/core/context_manager.py`. The prompt is assembled from a stable runtime prefix, working/durable memory, available skills, relevant memory retrieval candidates, session history, and the current request. Section budgets and floors are enforced there; the current request is not clipped, while relevant memory, skills, history, memory, and prefix are reduced in that order when the prompt exceeds budget.

Tools are explicitly registered in `bunnybyte/tools/registry.py` using the `RegisteredTool` abstraction from `bunnybyte/tools/base.py`. The registry is the model-visible capability whitelist: base tools include file listing, reading, search, shell execution, file write, exact patching, todos, subagents, plan mode, and ask-user. Before a tool runs, `tool_executor.run_tool()` validates arguments, blocks repeated identical calls, checks approval/permissions, checks tool policy, snapshots the workspace around risky tools, records affected paths and diff summaries, emits session events, and stores large shell outputs as run artifacts.

Subagents are session-scoped workers managed by `bunnybyte/core/worker_manager.py`. Workers run child runtimes built from the parent runtime; `Explore` is read-only style exploration, while worker tasks can receive write scopes. Background execution is used when a model client factory is available, and notifications are drained back into the main engine/TUI.

Memory and skills live under `bunnybyte/features/`. `memory.py` manages working memory, daily logs, durable topic indexes, memory tags, and auto-dream consolidation gates. `skills.py` discovers bundled, user, and project skills from `~/.bunnybyte/skills`, `skills/`, and `.bunnybyte/skills/`, parses skill frontmatter, renders prompt-visible skill summaries, and supports slash-command invocation through the skills runtime.

Shell sandboxing is implemented in `bunnybyte/features/sandbox/`. Sandbox settings are resolved from config and CLI, and `SandboxRunner` either runs commands plainly or wraps them with bubblewrap when available and required or requested. Workspace write access, extra read-only paths, denied paths, backend selection, and excluded commands are modeled in sandbox config.

Slash commands are defined centrally in `bunnybyte/commands/slash.py` and handled from CLI/TUI paths. Commands cover session management, runtime status, memory, plan mode, skills, subagents, provider/model switching, and usage reporting.

Tests are under `tests/` and are mostly pytest acceptance/unit tests around runtime behavior. `tests/test_runtime.py` and `tests/test_engine_acceptance.py` exercise the real session/event/tool loop with `ScriptedModelClient`. Provider/config/sandbox/memory/skills/tool-policy/TUI/evaluation behavior has targeted test files. `tests/test_architecture_boundaries.py` enforces line-count budgets for core modules, so large changes to central runtime files should usually be split into focused modules rather than growing those files indefinitely.

## Configuration and local data

Project provider configuration is expected in `.bunnybyte.toml`, copied from `.bunnybyte.toml.example`. Real API keys should not be committed; `.env.example` is documented as a legacy fallback and the README says to prefer `.bunnybyte.toml` for provider profiles.

Runtime-generated local data is stored under `.bunnybyte/`, including `sessions/<id>.json`, `sessions/<id>.events.jsonl`, `runs/<run_id>/`, memory indexes/logs/topics, project skills, and plan artifacts. These files are runtime evidence and memory rather than source architecture.

---
> Source: [LISAooo1234/Bunny-Byte](https://github.com/LISAooo1234/Bunny-Byte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
