## aibo

> Because your training data is out of date, you must **never** assume anything about models. You must look up model information online. Your first and most reliable source for general information is models.litellm.ai

# Repository Guidelines

## LLM parameters source of truth
Because your training data is out of date, you must **never** assume anything about models. You must look up model information online. Your first and most reliable source for general information is models.litellm.ai 

## Project Structure & Module Organization
- engine/: core evaluation logic, rate limiting, analytics, and input/output helpers.
- prompt_analyzer/: FastAPI migration work including the `web/` HTTP layer, shared `services/`, and Jinja2 assets.
- cli/: batch automation scripts that build workbooks, run experiments, and assemble reports. Should be only a frontend to engine/.
- prompt_analyzer/web/: FastAPI UI, routes, templates, and HTMX fragments that power the new web experience.
- tests/ mirrors engine modules with test files, and benchmarks/ stores curated evaluation datasets.
- docs/ captures architecture notes
- experiments/ archives benchmark runs and generated reports

## Build, Test, and Development Commands
- This project uses [uv](https://docs.astral.sh/uv/) for environment and dependency management; the lockfile is `uv.lock` and dependencies live in `pyproject.toml` (requires Python >=3.10).
- uv sync --extra web --extra dev creates/updates the `.venv` and installs runtime plus the web (FastAPI/Jinja/uvicorn) and dev (ruff/mypy/coverage) extras. Omit the extras for a runtime-only install.
- uv run uvicorn prompt_analyzer.web.app:create_app --factory --reload launches the FastAPI UI at http://127.0.0.1:8000 (see run-fastapi.ps1 for convenience on Windows).
- uv run cli/run.py --help lists batch and benchmark options that feed artifacts into experiments/.
- uv run pytest tests/ -v runs the full suite; add -m "not slow" for offline checks during development.
- Prefix one-off commands with `uv run` so they execute inside the managed environment without manual activation.

## Testing Guidelines
- Pytest markers slow, live_api, and integration gate external calls; apply them consistently when adding networked tests.
- Name files as test_feature.py and functions as test_scenario to align with pytest.ini discovery rules.

## Configuration & Security
- Load secrets through the .env file or the FastAPI UI sidebar; never commit keys, filled benchmark exports, or logs containing responses.
- When adding environment variables, update onboarding templates and document required reload steps in README.md.

## Unicode & Encoding (Windows Compatibility)
LLM responses commonly contain Unicode characters (arrows →, subscripts ₂, emoji, etc.) that Windows' default console encoding (cp1252) cannot display. This causes `UnicodeEncodeError` crashes that masquerade as "authentication failures" or other errors.

**Rules:**
- **NEVER use `print()` for any content that could originate from LLM responses or user input.** Use `logger.info()` instead—the logging handlers are configured with UTF-8 encoding.
- When writing files, always specify `encoding='utf-8'` explicitly.
- If you must use print() for debugging, wrap it in try/except for UnicodeEncodeError.
- The `engine/log_love.py` module handles print-to-logger redirection with Unicode sanitization, but new code should not rely on this—use logging directly.

---
> Source: [wharton-generative-ai-labs/AIBO](https://github.com/wharton-generative-ai-labs/AIBO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
