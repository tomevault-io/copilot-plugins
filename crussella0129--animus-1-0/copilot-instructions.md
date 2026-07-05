## animus-1-0

> Project-level guidance for Claude Code agents working on this repository. The user's global `~/.claude/CLAUDE.md` still applies; this file is additive and takes precedence on conflicts.

# CLAUDE.md — Animus

Project-level guidance for Claude Code agents working on this repository. The user's global `~/.claude/CLAUDE.md` still applies; this file is additive and takes precedence on conflicts.

## North Star

Animus is a **local-first agentic coding tool** powered by small local LLMs via llama-cpp-python. The capability ladder we're building toward:

1. **Simple ops, 100% accurate.** Move a file, rename a folder, mkdir, copy, delete — these must never be wrong. Small models miscall `mv`/`rm` when routed through `bash`, so each of these belongs as a dedicated tool with structured args and workspace boundary checks.
2. **Multi-step construction.** Given a prompt like "build me a CLI that does X," Animus should: deconstruct the task into elementary components → work each sequentially with appropriate tools → reconstruct the whole → devise and run tests. The existing `src/core/planner.py` is the seed but is **not yet wired into the runtime** — that's queued work.

Every new feature should advance one of those two rungs. If a change doesn't help simple-op reliability or multi-step construction, it's probably not for Animus.

## Critical principle: validate against a real GGUF

The unit tests mock the provider. They missed three real bugs in the v2-polish PR:

1. Installed CLI entry point bypassed Typer.
2. NANO `grammar_mode="full"` wedged the ReAct loop — model could never produce a final text answer.
3. Most GGUFs lack `general.parameter_count` metadata, collapsing tier detection to NANO.

**Never merge a change that touches the runtime, providers, grammar, tier behavior, or tool dispatch without a real-GGUF run.** Use `animus --debug` to capture a full trace. Inspect `.animus/trace/<session-id>.jsonl`. The user keeps GGUFs at `C:\Users\charl\.animus\models\`.

## Architectural conventions

- **Provider protocol** (`src/providers/base.py`) abstracts LLM backends. Tool-call recovery from raw model output lives in `src/providers/parsing.py` and is provider-agnostic — small models emit JSON in fenced blocks, `<tool_call>` tags, embedded prose, etc.; the shared parser handles all of those.
- **Tool registration** is declarative. Add tools by appending to `src/tools/defaults.py:register_default_tools`. Each tool is `(args: dict, workspace: Workspace) -> ToolResult` with a JSON schema, permission level, and min tier.
- **Tier system** (`src/core/tiers.py`) scales grammar enforcement, planner usage, max turns, and tool count to model size. NANO and SMALL use `grammar_mode="first_turn"` — never use `"full"` again; it makes loop termination impossible.
- **Workspace boundary** is enforced at every tool entry point via `workspace.resolve(path)`. Never bypass.
- **Observability**: the ReAct loop emits structured events (turn_start, iteration_start, provider_response, tool_call, tool_result, etc.) via `src/observability/tracer.py`. `animus --debug` writes JSONL + Rich live view; the JSONL is the source of truth and the pretty view is derived. **When adding runtime features, emit events for them** — it's how we'll catch the next real-model loop pathology.

## Conventions for new tools

When you add a new tool (e.g. `move_file`, `mkdir`):

1. Handler in `src/tools/<area>.py`: signature `def handle_X(args, workspace) -> ToolResult`. Resolve every path through `workspace.resolve()`. Catch `OSError` and return `ToolResult(output=..., is_error=True)`.
2. Register in `src/tools/defaults.py` with a JSON schema, `PermissionLevel`, and `min_tier`. Simple ops (move, rename, mkdir) should be `NANO`-tier minimum so all model sizes can use them.
3. Add a test file `tests/test_tools_<area>.py` exercising: happy path, workspace-escape path, missing-source path, OSError path.
4. Run the suite (`pytest`) **and** a real-GGUF run with `--debug` against a small workspace that uses the new tool.

## Conventions for runtime/provider changes

- Keep the runtime's per-iteration cost low. Small models on CPU regenerate the whole prompt every turn; anything you do per-iteration multiplies. Prefer cache-once-per-turn (see `cached_grammar` in `runtime.py`).
- Tool outputs are truncated to `config.session.max_tool_output_chars` before they re-enter the prompt. The **debug trace sees the full, untruncated output** — emit the tracer event before the truncation, not after.
- The `Provider.estimate_tokens` protocol method should use the real tokenizer when available; only fall back to chars/4 if it raises. Inaccurate counts over-fill small context windows.

## Out of scope

- Audio / TTS / voice. The v2 rewrite already removed these; do not re-add.
- Cloud/HTTP providers. Local-first is the point.
- Anything that depends on internet at runtime (huggingface-cli installs are fine ahead of time; runtime model loading is purely local).

## Open follow-on work (ranked)

1. **Wire planner into runtime.** `src/core/planner.py` exists but isn't called. With Phase 7 (task_complete + tool summary + full grammar), the SMALL tier already handles short multi-step prompts natively. The planner becomes valuable for *strategic* decomposition — "build me a CLI that does X" → [design files] → [write code] → [tests] → [iterate] — where each phase has a scoped tool set.
2. **`run_tests` tool.** Detect language, invoke the right test runner, return structured pass/fail. Required for the test-and-iterate part of construction.
3. **Structured compaction.** Current compactor produces a lossy text summary. For long construction sessions, preserve tool-history structurally so the model can revisit what it built.
4. **Default-prompt migration tip.** Users with an old `~/.animus/config.yaml` from pre-v2.1 have a stale `system_prompt` that overrides the new default (the one that teaches task_complete). If the model on a fresh workspace ignores task_complete, that's the cause — either clear the stale `system_prompt` line in their global config or override it in the workspace's `.animus/config.local.yaml`.

## What's done (Phase 7 of PR #3)

- File-op tools: `move_path`, `copy_path`, `delete_path` (with `recursive` safety flag), `make_dir`. NANO-tier, WRITE permission, workspace-boundary-checked.
- `task_complete(summary)` meta-tool — structured loop terminator. Registered as a regular `ToolSpec` but intercepted in the runtime before dispatch. Enables `grammar_mode="full"` for NANO and SMALL.
- `ToolRunner.summary_for(tier, policy)` builds a signature-style tool summary that the runtime prepends to the system prompt every turn. Filtered by tier + permission. Solves "model knows the grammar but not the semantics."

---
> Source: [crussella0129/Animus_1.0](https://github.com/crussella0129/Animus_1.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
