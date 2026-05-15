## skim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make install     # Install deps + set up git hooks
make test        # Run all tests
make lint        # Ruff check
make format      # Ruff format
uv run pytest tests/test_cache.py::test_get_cached_miss -v  # Single test
uv run skim --help   # Test CLI locally (picks up source changes immediately)
```

Git hooks (`.githooks/`): pre-commit runs ruff format+check, pre-push runs `uv run python -m pytest`. Set up automatically by `make install`. Note: bare `pytest` fails — always use `python -m pytest`.

## Architecture

**CLI tool** that takes arxiv paper IDs, downloads PDFs, sends them to an LLM, and generates summaries in two flavors: `story` (layperson narrative) and `deep` (technical summary).

### Data flow

```
arxiv ID → arxiv.py (download PDF) → llm.py (dispatch to backend) → cache.py (save) → cli.py (render with rich) → viewer.py (optional: --open for browser with KaTeX)
```

### Key design decisions

- **Prompts are bundled in the package** at `src/skim/prompts/` and loaded via `importlib.resources` — not from the filesystem. This is what makes `uv tool install git+...` work.
- **Cache invalidation uses prompt hashes.** Cache filenames include the first 8 chars of the prompt's SHA-256: `{arxiv_id}_{type}_{hash}.md`. Changing a prompt automatically invalidates all cached results for that type.
- **Two backends:** `openai` (any OpenAI-compatible API, sends PDF as base64 file content block) and `claude` (calls `claude -p` via subprocess, uses Read tool for PDF — no API key needed). Backend is selected during `skim init`.
- **Config lives at `~/.config/skim/config.toml`** (respects `$XDG_CONFIG_HOME`). No `.env` files. The `Config` dataclass has: `backend`, `api_key`, `base_url`, `model`, `output_dir`.
- **Two summary types** map to prompt files: `story` → `story.md`, `deep` → `deep.md`. The mapping is `PROMPT_MAP` in `llm.py`.
- **`cli.py` uses argparse with subparsers** for `init` and `cd`, plus top-level flags `-p`/`-t`/`--output-dir` for the main summarize flow.
- **LaTeX → Unicode conversion** (`latex.py`) is applied only for terminal display, not saved files. Uses negative lookahead patterns for short commands (`\in`, `\pi`) to avoid prefix corruption.

### Module dependency graph

```
cli.py → init_cmd.py → config.py
cli.py → llm.py → config.py
llm.py → claude_backend.py (lazy import, only when backend == "claude")
cli.py → arxiv.py
cli.py → cache.py
cli.py → latex.py
cli.py → viewer.py (browser viewer with KaTeX, only when --open is used)
```

`arxiv.py`, `cache.py`, and `latex.py` are standalone — no internal imports.

## Workflow

- When changing anything, always consider impact on all three usage modes: CLI+OpenAI, CLI+Claude backend, and Claude Code plugin. Test or add tests for all three where applicable.
- **Versioning (always prompt the user if they forget):**
  - Patch (`0.x.Y`) for any minor change (bug fix, docs, styling).
  - Minor (`0.X.0`) for new features or slightly major changes.
  - Major (`X.0.0`) for final/breaking releases.
  - Always grep the repo for the old version string to find all locations that need updating.

## Gotchas

- `uv sync` required after switching branches — stale installs cause confusing errors (e.g., old config validation running despite new code).
- Never add `Co-Authored-By: Claude` to commits.
- Git hooks block commits/pushes on lint/test failures. Run `make format` before committing.
- When adding new fields to `Config` dataclass, update: `config.py`, `init_cmd.py`, `save_config()`, all test files that construct `Config(...)`.
- When bumping version in `pyproject.toml` and `__init__.py`, also update: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `tests/test_cli.py::test_main_version`.
- Claude Code plugin cache (`~/.claude/plugins/cache/skim/`) is keyed by version. Bumping the version in `plugin.json` + `marketplace.json` is required to force users to get updated skills. Uninstall + reinstall is not enough without a version bump.
- Skills in `skills/*/SKILL.md` must only use `name` and `description` in YAML frontmatter. Other fields (like `allowed-tools`, `argument-hint`) break skill registration silently.
- When embedding JavaScript regex in Python `.format()` templates, do regex/string processing in Python instead. Backslash escaping between Python string literals → `.format()` → JavaScript is extremely error-prone.
- LLM output can contain UTF-8 surrogates. When writing to files, use binary mode with `.encode("utf-8", errors="replace")` to avoid `UnicodeEncodeError`.

---
> Source: [dipta007/skim](https://github.com/dipta007/skim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
