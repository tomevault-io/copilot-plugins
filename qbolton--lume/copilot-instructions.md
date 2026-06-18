## lume

> - Use an explanatory style: explain reasoning, decisions, and trade-offs — not just what, but why.

# Output Style (explanatory)

- Use an explanatory style: explain reasoning, decisions, and trade-offs — not just what, but why. 
- Clearly highlight these sections of output 

# Project Rules
- Always use the virtual environment located at `.venv`
- All shell commands requiring Python should be run within the virtual env.
- Update the user regularly at each step
- Always simplify code wherever possible.  Use code-simplifier during pre-commit for best results.

# Python Package Management with uv

Use uv exclusively for Python package management in this project.

## Package Management Commands

- All Python dependencies **must be installed, synchronized, and locked** using uv
- Never use pip, pip-tools, poetry, or conda directly for dependency management

Use these commands:

- Install dependencies: `uv add <package>`
- Remove dependencies: `uv remove <package>`
- Sync dependencies: `uv sync`

## Running Python Code

- Run a Python script with `uv run <script-name>.py`
- Run Python tools like Pytest with `uv run pytest` or `uv run ruff`
- Launch a Python repl with `uv run python`

## Managing Scripts with PEP 723 Inline Metadata

- Run a Python script with inline metadata (dependencies defined at the top of the file) with: `uv run script.py`
- You can add or remove dependencies manually from the `dependencies =` section at the top of the script, or
- Or using uv CLI:
    - `uv add package-name --script script.py`
    - `uv remove package-name --script script.py`

# Project Structure

- Source code: `src/lume/` (package layout with uv_build)
- Pipeline stages: `src/lume/stages/` (preprocessor, analyzer, scorer, rules)
- LLM integration: `src/lume/llm/` (isolated from deterministic logic)
- Niche configs: `configs/niches/*.json` (camelCase keys, converted at load time)
- App settings: `configs/settings.toml` (loaded by `load_settings()` in `config.py`)
- Tests: `tests/` (mirror source structure, run with `uv run pytest tests/ -v`)
- Docs: `docs/plan.md` has the full architecture plan

# Coding Conventions

- All data flows between pipeline stages use Pydantic models defined in `models.py`
- Pipeline stages are plain functions, not classes — promote to classes only when state is needed
- Prefer stdlib over external deps for simple tasks (e.g. `html.parser` over beautifulsoup)
- Tests are written alongside implementation, not after
- Use `datetime.now(UTC)` not `datetime.utcnow()` (deprecated in 3.12)
- Functions suffixed or parameterized with `text_lower` expect pre-lowered input; lowercase once at the entry point, not in every helper
- Signal indicators use hybrid resolution: niche-defined `signalIndicators` in the JSON config take priority over the built-in
`SIGNAL_INDICATORS` dict in `analyzer.py`. Add niche-specific signals to the config, not to the built-in dict.
- Config resolution uses `LUME_CONFIG_DIR` env var (priority 1), then repo-relative `configs/` (priority 2). `PROJECT_ROOT` was intentionally removed — don't re-add it.

# LLM Integration Conventions

- LLM code lives exclusively in `src/lume/llm/` — never import litellm or LLM logic into deterministic stages (scorer, rules, preprocessor)
- The analyzer uses **late imports** for `lume.llm.*` so litellm doesn't load in keyword-only mode. Maintain this pattern — don't move LLM imports to the top of `analyzer.py`
- `fetch.py` uses a **late import** of `load_settings()` from `config.py` to resolve the default User-Agent. Same convention — don't move it to the top level.
- When mocking LLM calls in tests, target `lume.llm.client.call_llm` (the source module), **not** `lume.stages.analyzer.call_llm` — late imports mean the name doesn't exist on the analyzer module
- Test helpers for deterministic pipeline behavior should set `llm_config=LLMConfig(enabled=False)` to avoid accidental real LLM calls and warnings

---
> Source: [qbolton/lume](https://github.com/qbolton/lume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
