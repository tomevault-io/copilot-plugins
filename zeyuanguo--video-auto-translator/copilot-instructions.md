## video-auto-translator

> Read this file first, then consult `.windsurf/rules/dev-basic-rules.md` for the full project development rules. That file is the detailed source for collaboration style, fail-fast behavior, config discipline, GPU expectations, and documentation hygiene. If this guide is brief and `.windsurf` is more specific, follow `.windsurf`.

# Repository Guidelines

## Contributor Workflow
Read this file first, then consult `.windsurf/rules/dev-basic-rules.md` for the full project development rules. That file is the detailed source for collaboration style, fail-fast behavior, config discipline, GPU expectations, and documentation hygiene. If this guide is brief and `.windsurf` is more specific, follow `.windsurf`.

## Project Structure & Module Organization
`vat/` contains the application code. Start with `vat/__main__.py` and `vat/cli/commands.py` for CLI behavior, then follow the processing flow in `vat/pipeline/`. Shared state and contracts live in `vat/config.py`, `vat/models.py`, and `vat/database.py`. The WebUI is in `vat/web/` with route handlers under `vat/web/routes/`; it invokes CLI subprocesses rather than bypassing the CLI layer. Default configuration lives in `config/default.yaml`. Tests are under `tests/`, and many subsystems include local notes such as `vat/downloaders/readme.md`.

## Build, Test, and Development Commands
`pip install -r requirements.txt` installs runtime dependencies.
`pip install -e .` installs the editable `vat` command for local development.
`vat init` creates a local config scaffold.
`vat --help` or `python -m vat --help` lists available workflows.
`vat web --port 8080` starts the FastAPI + Jinja2 management UI.
`pytest` runs the full suite; use `pytest tests/test_pipeline.py -q` or another focused file for faster iteration.

## Coding Style & Naming Conventions
Use 4-space indentation and keep Python code explicit, readable, and fail-fast. Do not duplicate config defaults across YAML, CLI flags, and hardcoded constants. Preserve the existing architecture: CLI is the core entrypoint, WebUI is a management layer on top. Follow Python naming norms: `snake_case` for modules/functions, `PascalCase` for classes, `UPPER_CASE` for constants, and `test_*.py` for test files. No repo-wide formatter or linter config is checked in, so match the surrounding style and keep diffs tight.

## Testing Guidelines
Use `pytest` for all regression coverage. Any meaningful behavior change should include or update tests. When touching pipeline stages or status semantics, cross-check `vat/models.py`, `vat/database.py`, `vat/pipeline/`, and the related web/job tests together. Prefer targeted unit tests first, then rerun a broader subset if the change crosses module boundaries.

## Commit & Pull Request Guidelines
Recent commits use short prefixes such as `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, and `refine:`. Keep commit messages imperative and scoped to one change. Pull requests should summarize user-visible impact, list affected commands or pipeline stages, link related issues, and include screenshots when changing `vat/web/templates/` or other UI behavior.

## Security & Configuration Tips
Keep API keys and local secrets out of git; prefer environment variables such as `VAT_LLM_APIKEY`. Treat `config/default.yaml` as the source of truth for supported settings, and fail loudly instead of silently falling back to CPU or alternate behavior when GPU or external service assumptions are not met.

---
> Source: [ZeyuanGuo/Video-Auto-Translator](https://github.com/ZeyuanGuo/Video-Auto-Translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
