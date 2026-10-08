## physics-intern-v0

> This is a multi-agent scaffolding system for autonomous scientific research in mathematics and theoretical physics.

# PhysicsIntern

This is a multi-agent scaffolding system for autonomous scientific research in mathematics and theoretical physics.

See README.md for overview, motivation, and design principles.

## Tech Stack

- Use `uv` for dependency management
- Tests: `pytest` (run with `uv run python -m pytest -v`, need `--extra testing`)
- `rich` for console output

## CI checks (must pass before declaring code work done)

GitHub Actions (`.github/workflows/testing.yml`) runs three commands on every push/PR. Run all three locally before reporting any code change as done — they are cheap (~25s total) and catch the same failures CI would:

```bash
uv run ruff check tests src scripts serve
uv run ruff format --check tests src scripts serve
uv run python -m pytest ./tests/
```

Notes:
- `ruff check` and `ruff format --check` run sequentially in CI, so a `ruff check` failure masks any pending format failure — always run both.
- If `ruff format --check` reports diffs, apply them with `uv run ruff format tests src scripts serve` rather than hand-editing.
- New module-level code that runs *before* imports (e.g. `load_dotenv()` between import lines) needs an `E402` per-file-ignore in `pyproject.toml` under `[tool.ruff.lint.per-file-ignores]` — see existing entries for runner/CLI files.

## Key Invariants

These are non-obvious rules enforced by the scaffolding. Violating them will break things.

- **`ResearchState` is the single source of truth** — agents render context from it via renderers. No file read-back from disk.
- **MD files are write-only snapshots** — `RESEARCH_STATE.md`, `EVIDENCE_LOG.md`, `CRITIQUE_LOG.md` are rendered once per iteration for git history and `verify.py`. Never read back by agents.
- **Fresh context per call** — every agent gets a new context built from `ResearchState`. No conversation history carries over.
- **Tool definitions use OpenAI canonical format** (`type: "function"`, `function: {name, description, parameters}`). The Anthropic adapter transforms to `input_schema` format.
- **Agent prompts are static `.md` files** co-located with each agent in `src/physics_intern/agents/<name>/prompt.md` — no templating.
- **YAML frontmatter parsing always falls back to regex** — never crash the loop on parse failure.
- **Workspace git is managed by the scaffolding loop**, not by agents.
- **ERs are immutable** — only the adjudicator can demote them (via valid critique). No direct dispatch to ERs.
- **WH→ER promotion is automatic** — `auto_promote` in `critique_routing.py` (called via `engine._auto_promote`) fires after VERIFIED review when dependencies are satisfied, with cascading.
- **Iteration counter is scaffolding-maintained** (`_update_research_iteration()`), not LLM-dependent.
- **`BaseAgent.tools` class attribute determines mode** — non-empty → agentic loop (`run_agent_loop`), empty → one-shot (`call_llm`). Both go through the provider abstraction layer.

### Autophysicist-specific

- **Single-agent, stateless iterations** — the Manager gets a fresh context each iteration. Memory is only what was written to PermanentMemory or Scratchpad.
- **PermanentMemory is append-only, Scratchpad is windowed** — permanent memory is fully visible every iteration; scratchpad shows only the last N entries (default: 5).
- **Token budget triggers wind-down** — when per-iteration tokens exceed `--token-budget`, `dispatch_subagent` is removed; at 1.5× budget the iteration is force-terminated.
- **`submit_final_answer` terminates the run** — sets `problem_solved = True` and breaks the iteration loop. Formal evaluation runs automatically after.

---
> Source: [huggingface/physics-intern-v0](https://github.com/huggingface/physics-intern-v0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
