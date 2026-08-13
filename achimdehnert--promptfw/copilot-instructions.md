## promptfw

> Operating guide for an AI agent working in this repo. Repo-specific; the

# CLAUDE.md — iil-promptfw

Operating guide for an AI agent working in this repo. Repo-specific; the
user-level `~/.claude/CLAUDE.md` still applies and wins on conflicts.

## What this is

`iil-promptfw` (dist `iil-promptfw`, import `promptfw`) is a **5-layer Jinja2
prompt template framework** for LLM applications. It registers `PromptTemplate`
objects across the layers `SYSTEM → FORMAT → CONTEXT* → TASK → FEW_SHOT`,
renders them through Jinja2 into a system + user prompt (`RenderedPrompt`), or
directly into an OpenAI/LiteLLM message list. It adds wildcard lookup, version
pinning, fallback chains, format-aware filtering, optional tiktoken token
estimation, YAML/frontmatter loading, hot reload, LLM-response parsers, and an
optional Django ORM adapter. Shipped as a pure-Python PyPI library (no CLI).

## Setup

```bash
python3 -m pip install -e ".[dev]"   # editable install with dev extras
```

`__version__` is read from installed package metadata (`promptfw.__version__`);
in a bare source checkout without an install it falls back to `0.0.0.dev0`.

## Test / lint / types

```bash
make test     # python3 -m pytest tests/ --tb=short -q   (251 tests)
make test-v   # same, verbose
make lint     # ruff check src/ tests/
make install  # pip install -e ".[dev]"
make clean    # remove __pycache__ + .pytest_cache
```

- `pytest` config lives in `[tool.pytest.ini_options]`: `pythonpath = ["src"]`,
  `asyncio_mode = "auto"`, `testpaths = ["tests"]`.
- There is no committed `[tool.mypy]` config and no `types` target — type
  checking is not part of the gate today.

## Architecture (module map)

| Module | Responsibility |
|---|---|
| `schema.py` | `PromptTemplate`, `RenderedPrompt`, `TemplateLayer`, `USER_LAYERS`, `to_messages()` |
| `registry.py` | `TemplateRegistry` — wildcard lookup, version pinning, fallback chains |
| `renderer.py` | `PromptRenderer` — Jinja2 rendering engine |
| `stack.py` | `PromptStack` — high-level facade (register, render, render_to_messages, hot reload) |
| `parsing.py` | `extract_json`, `extract_json_list`, `extract_json_strict`, `extract_field`, `<think>` stripping |
| `frontmatter.py` | `render_frontmatter_file` / `render_frontmatter_string` (YAML frontmatter + body) |
| `planning.py` | `get_planning_stack()` + `PLANNING_TEMPLATES` |
| `writing.py` | `get_writing_stack` / `get_academic_writing_stack` / `get_scientific_writing_stack` |
| `lektorat.py` | `get_lektorat_stack()` + `LEKTORAT_TEMPLATES` |
| `concept_analysis.py` | `get_concept_analysis_stack()` + `CONCEPT_ANALYSIS_TEMPLATES` |
| `db_resolver.py` | `DBPromptResolver` — DB-backed template resolution |
| `django_registry.py` | `DjangoTemplateRegistry`, `BFAGENT_FIELD_MAP` — ORM adapter |
| `contrib/django/` | optional Django app: models, admin, migrations, management commands |
| `exceptions.py` | `TemplateNotFoundError`, `TemplateRenderError`, `LLMResponseError` |

The public API surface is re-exported from `promptfw/__init__.py` (`__all__`).

## Conventions

- Commits: `[feat|fix|refactor|docs|test|chore](scope): description`.
- Tests: `test_should_<expected_behavior>` (a pytest plugin nudges this; legacy
  test names still exist).
- Optional features degrade gracefully: `tiktoken`, `watchdog` (hotreload) and
  `django`/`pydantic` are extras — code must not hard-import them at module top
  level outside `contrib/`.

## Release (gated — not on merge)

Versioned in `pyproject.toml` + `CHANGELOG.md` (Keep a Changelog). Publishing to
PyPI is a **deliberate, gated step** via `.github/workflows/publish.yml`
(triggered by a `v*` tag push or `workflow_dispatch`) — never automatic on
merge. Keep `pyproject.version`, the CHANGELOG top entry, and the published
PyPI version in sync. Do not tag/publish without an explicit go-ahead.

## Known issues / gotchas

- **PyPI publish drift:** `pyproject` is at `0.8.1` but it is not published; the
  `__init__.py` literal previously lagged at `0.7.0` (now resolved from metadata
  instead of a literal). See `AGENT_HANDOVER.md`.
- No `[tool.mypy]` config and no coverage floor configured.
- See `AGENT_HANDOVER.md` for current state and next priorities.

---
> Source: [achimdehnert/promptfw](https://github.com/achimdehnert/promptfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
