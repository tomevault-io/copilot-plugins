## watch-my-escape

> 'WATCH MY ESCAPE' is an application where an LLM tries to escape a man-made puzzle room.

# WATCH MY ESCAPE

## Project Description

'WATCH MY ESCAPE' is an application where an LLM tries to escape a man-made puzzle room.

## Project Structure

- `src/watch_my_escape/app/` contains Gradio/FastAPI app wiring and API schemas.
- `src/watch_my_escape/game/` contains escape-room state, actions, runtime rules, and map loading.
- `src/watch_my_escape/game/map_data/` contains built-in JSON room maps.
- `src/watch_my_escape/agent/` contains LLM agent orchestration, prompts, runs, and emotions.
- `src/watch_my_escape/agent/prompt_templates/` contains Markdown prompt templates.
- `src/watch_my_escape/llm/` contains model configuration, provider integration, tracing, evaluation, and tool-call parsing.
- `src/watch_my_escape/web/` contains Jinja2 templates and source frontend assets.
- `src/watch_my_escape/web/static/app/` contains browser-side application modules.
- `src/watch_my_escape/__main__.py`, `doctor.py`, and `setup_llm.py` contain package entry points and helper commands.
- `build/web/static/` contains generated frontend assets from `make assets`.
- `requirements/` contains platform-specific LLM dependency inputs.
- `space/` contains the Hugging Face Space deployment entry point and requirements.
- `tests/` contains pytest tests for backend behavior.
- `pyproject.toml` defines dependencies, Ruff linting, pytest settings, package metadata, and console scripts.
- `.pre-commit-config.yaml` defines Git pre-commit hooks that run formatting, linting, type checking, and tests.
- `package.json` and `package-lock.json` define frontend build dependencies.
- `.github/workflows/` contains deployment automation.
- `Makefile` provides the common development commands, including asset builds, linting, tests, app launch, doctor, and LLM setup helpers.

## Development Workflow

- Always use modern Python practices for Python 3.12+.
- Use TDD where appropriate to keep a considered design and protect key behaviours.
- Tidy-up and refactor after changes - make sure to follow SOLID principles.
- Run `make lint` and `make test` after changes.
- Use `uv run` for Python commands.
- Do not commit changes unless instructed.
- Do not start the web server unless instructed.

## Comments

- Keep all comments concise, clear, and suitable for inclusion in final production.
- Only use comments when the intent cannot be explained through thoughtful naming or code structure.

## Attribution

- When committing Codex-assisted work, include this trailer in the commit message:
  `Co-authored-by: Codex <codex@openai.com>`

---
> Source: [cjami/watch-my-escape](https://github.com/cjami/watch-my-escape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
