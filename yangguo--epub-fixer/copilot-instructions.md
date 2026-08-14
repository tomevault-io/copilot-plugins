## epub-fixer

> Core entry points (`epub_master_fixer.py`, `epub_agent_cli.py`, `validate_epub.py`) live in the repo root for quick CLI access. Shared configuration and helpers sit in `config.py` and `utils.py`. Agent-specific orchestration, validation, fixing, DRM, and LLM code is grouped under `agent_system/`, while reference docs (such as `docs/LLM_GUIDE.md`) and legacy assets (`archive/`) remain isolated from production code. Keep EPUB samples and generated artifacts out of version control unless they illustrate a failing test case.

# Repository Guidelines

## Project Structure & Module Organization
Core entry points (`epub_master_fixer.py`, `epub_agent_cli.py`, `validate_epub.py`) live in the repo root for quick CLI access. Shared configuration and helpers sit in `config.py` and `utils.py`. Agent-specific orchestration, validation, fixing, DRM, and LLM code is grouped under `agent_system/`, while reference docs (such as `docs/LLM_GUIDE.md`) and legacy assets (`archive/`) remain isolated from production code. Keep EPUB samples and generated artifacts out of version control unless they illustrate a failing test case.

## Build, Test, and Development Commands
- `pip install -r requirements.txt` — install Python and Anthropic dependencies.
- `python epub_master_fixer.py book.epub` — run the deterministic fixer; creates `*_backup.epub` automatically.
- `python epub_agent_cli.py book.epub --llm --workflow validation fixing validation` — execute the agent pipeline with Claude-enabled reasoning.
- `python validate_epub.py book.epub` — thin wrapper around `epubcheck.jar` for smoke validation.
- `python test_syntax.py` — lightweight import/syntax gate for agent modules; run before every PR.

## Coding Style & Naming Conventions
Target Python 3.10+, adhere to 4-space indentation, and follow PEP 8/PEP 257. Modules and functions use `snake_case`, classes follow `PascalCase`, and constants stay uppercase. Prefer explicit type hints in public APIs, short helper functions, and f-strings for logging. Keep Markdown docs <100‑character lines for readability.

## Testing Guidelines
Integration tests rely on real EPUB fixtures; add minimal samples under `archive/` or a temp path, not `agent_system/`. Expand `test_syntax.py` or add new `tests/` modules when logic grows. Name tests after the feature under test (`test_fixing_agent_handles_missing_body`). Always run `python test_syntax.py` plus any new pytest suites before shipping, and attach failing epubcheck excerpts in the PR description when bugs persist.

## Commit & Pull Request Guidelines
Write imperative, scope-prefixed commits (e.g., `fix: normalize manifest ids`) and keep them under ~75 characters. Each PR should describe the motivation, list manual/automated tests, and reference related issues. Include reproduction steps for EPUB regressions, mention whether Claude/epubcheck were exercised, and add screenshots or log excerpts when fixing agent workflows. Request review from another agent contributor before merging.

## Agent-Specific Tips
Store sensitive keys in `.env` (see `docs/LLM_GUIDE.md`) and never commit them. When modifying `llm_brain.py`, add logging that clarifies model/endpoint choices so multi-agent runs remain debuggable. Preserve the contract between orchestrator steps (`validation → fixing → validation`) unless the README workflow guidance is updated in tandem.

---
> Source: [yangguo/epub_fixer](https://github.com/yangguo/epub_fixer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
