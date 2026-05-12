## rlm-dspy

> - Root scripts: `main.py` demonstrates the needle-in-a-haystack flow; keep new entry points here or under `examples/` if you add one. The script now prints both the human-readable answer and any `FINAL_VAR` payload returned via the new `RLMResult` container.

# Repository Guidelines

## Project Structure & Module Organization
- Root scripts: `main.py` demonstrates the needle-in-a-haystack flow; keep new entry points here or under `examples/` if you add one. The script now prints both the human-readable answer and any `FINAL_VAR` payload returned via the new `RLMResult` container.
- Core package: `rlm/` hosts the reusable modules. `rlm/rlm_repl.py` implements the recursive REPL agent, `rlm/repl.py` wraps the exec-based environment, and `rlm/utils/` houses prompt templates, OpenAI client, and execution helpers. `rlm/logger/` contains optional colorful logging.
- Assets: `media/` stores illustrative images. Avoid mixing generated logs or datasets here.

## Build, Test, and Development Commands
- Install deps once with `pip install -r requirements.txt`.
- Run the sample workflow via `python main.py`; adjust `RLM_REPL` constructor flags (model names, recursion depth) directly in that file when experimenting.
- For quick module checks, prefer `python -m rlm.rlm_repl` style invocations so imports resolve relative to the package.

## Coding Style & Naming Conventions
- Follow PEP 8: 4-space indents, snake_case for functions/variables, PascalCase for classes, and keep lines under 100 chars as in the existing modules.
- Maintain type hints and docstrings; new helper functions should mirror the concise docstring + inline comment style already present.
- Group related utilities under `rlm/utils/` and expose clean entry points in `__init__.py` when needed.

## Testing Guidelines
- No automated suite ships today; when adding features, create lightweight scripts under `tests/` or `examples/` that prove the behavior and run them with `python -m pytest` (preferred) or direct module execution.
- Keep long-context fixtures small (≤1e4 lines) for routine verification, and reserve million-line runs for targeted benchmarks.
- Document any required environment flags (e.g., recursion depth, logging) in the test script docstring.

## Commit & Pull Request Guidelines
- Use short, imperative commit messages similar to `Update README.md` and `Minimal working version`; keep scope focused per commit.
- PRs should summarize purpose, outline testing done (commands + inputs), link issues when applicable, and attach screenshots or logs if logging output changed.
- Flag any API or prompt contract changes prominently so downstream agents can adjust.

## Configuration & Logging
- Set `OPENAI_API_KEY` in your shell or `.env`; the OpenAI client raises early if missing.
- Toggle rich logging by passing `enable_logging=True` to `RLM_REPL`; the colorful logger expects the optional `rich` package already listed in `requirements.txt`.

---
> Source: [halfprice06/rlm_dspy](https://github.com/halfprice06/rlm_dspy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
