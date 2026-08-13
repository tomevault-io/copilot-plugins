## pm-notes

> This repository is a mixed knowledge base plus runnable examples.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a mixed knowledge base plus runnable examples.
- `ai-dt/`: AI/DT study notes (RAG, MCP, OpenSearch).
- `web-development/`: web learning notes and sample projects.
- `dev-environment/`: terminal/tooling notes.
- `Codes/python/`: executable Python examples (`history-opensearch`, `knowhow-elasticsearch`, `opensearch_handler`, etc.).
- `web-development/python/flask/job-scheduler/`: the most complete app (Flask backend + Nuxt frontend).

Keep docs close to their topic (`README.md` in each module), and keep runnable code within its module folder.


## Coding Style & Naming Conventions
- Python: 4-space indentation, `snake_case` functions/variables, `PascalCase` classes, type hints for new/changed code.
- Vue/TS (Nuxt): follow ESLint + Nuxt defaults; use 2-space indentation (see `frontend/.editorconfig`).
- Markdown: concise sections, clear headings, and relative links.
- File/folder names: use descriptive `kebab-case` or `snake_case`; match existing local conventions in each module.

## Testing Guidelines
Automated tests are not yet standardized across the repository. For code changes:
- Run lint/type-check where available (Nuxt frontend).
- Perform module-level smoke tests (run the affected script/app end-to-end).
- When adding non-trivial logic, add a local `tests/` folder in that module and prefer `pytest` with `test_*.py` naming.

## Commit & Pull Request Guidelines
Recent history follows short, imperative commit subjects (for example: `Add ...`, `Refactor ...`, `Update ...`, `Clean up ...`).
- Keep subject lines concise and action-first.
- One logical change per commit.
- PRs should include: purpose, affected paths, commands run for validation, and screenshots for UI changes (`job-scheduler/frontend`).

## Security & Configuration Tips
- Do not commit secrets, API keys, or internal endpoints.
- Keep environment-specific values in local config overrides or environment variables.
- Review `config.py`/`config.yaml` changes carefully before opening a PR.

---
> Source: [DarrenKoi/pm_notes](https://github.com/DarrenKoi/pm_notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
