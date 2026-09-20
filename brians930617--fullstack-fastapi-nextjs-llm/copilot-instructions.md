## fullstack-fastapi-nextjs-llm

> Guidance for AI coding agents (Codex, Copilot, Cursor, Zed, OpenCode) working with this repository.

# AGENTS.md

Guidance for AI coding agents (Codex, Copilot, Cursor, Zed, OpenCode) working with this repository.

## Project Overview

**fastapi-fullstack** is an interactive CLI tool that generates FastAPI projects with Logfire observability integration. It uses Cookiecutter templates to scaffold complete project structures with configurable options for databases, authentication, background tasks, and various integrations.

## Commands

```bash
# Install dependencies
uv sync

# Run tests
pytest

# Run a single test file
pytest tests/test_cli.py -v

# Linting and formatting (always run before committing)
ruff check . --fix
ruff format .

# Type checking
mypy fastapi_gen
```

## CLI Usage

```bash
# Interactive wizard
fastapi-fullstack new

# Quick project creation
fastapi-fullstack create my_project --database postgresql --auth jwt

# Minimal project (no extras)
fastapi-fullstack create my_project --minimal
```

## Architecture

### Core Modules (`fastapi_gen/`)

| Module | Purpose |
|--------|---------|
| `cli.py` | Click-based CLI with commands: `new`, `create`, `templates` |
| `config.py` | Pydantic models for configuration options |
| `prompts.py` | Interactive prompts using Questionary |
| `generator.py` | Cookiecutter invocation and messaging |

### Template System (`template/`)

```
template/
├── cookiecutter.json                    # Default context (~75 variables)
├── hooks/post_gen_project.py            # Post-generation cleanup
└── {{cookiecutter.project_slug}}/
    ├── backend/app/                     # FastAPI application
    └── frontend/                        # Next.js 15 (optional)
```

Template files use Jinja2 conditionals: `{% if cookiecutter.use_jwt %}...{% endif %}`

## Key Design Decisions

- **Async-first**: All database options except SQLite use async drivers (asyncpg, motor)
- **Naming**: Project names must match `^[a-z][a-z0-9_]*$`
- **Package management**: Generated projects use UV
- **Data access**: Repository pattern throughout

## Testing Guidelines

- Tests live in `tests/` and mirror the `fastapi_gen/` structure
- Use `pytest` fixtures from `conftest.py` for common setup
- Template generation tests should use temporary directories
- Run the full test suite before submitting changes

## Common Tasks

**Adding a new CLI option:**
1. Add the option to `config.py` (Pydantic model)
2. Add the prompt to `prompts.py`
3. Update `cookiecutter.json` with the new variable
4. Add conditional logic to relevant template files

**Modifying template output:**
1. Edit files under `template/{{cookiecutter.project_slug}}/`
2. Use `{% if cookiecutter.variable %}` for conditional content
3. Test with `fastapi-fullstack create test_project --<relevant-flags>`
4. Check `hooks/post_gen_project.py` if files need conditional deletion

## Reference

| Resource | Location |
|----------|----------|
| Template variables | `template/cookiecutter.json` |
| Post-generation hooks | `template/hooks/post_gen_project.py` |
| Sprint planning | `notes/sprint_0_1_7/` |
| CLI help | `fastapi-fullstack --help` |

---
> Source: [brians930617/fullstack-fastapi-nextjs-llm](https://github.com/brians930617/fullstack-fastapi-nextjs-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
