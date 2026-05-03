## python-dtctl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **⚠️ DISCLAIMER**: This tool is provided "as-is" without warranty and is **not produced, endorsed, or supported by Dynatrace**. It is an independent, community-driven project. **Use at your own risk.** The authors assume no liability for any issues arising from its use. Always test in non-production environments first. For official Dynatrace tools and support, please visit [dynatrace.com](https://www.dynatrace.com).

## Instructions Location

Claude Code instructions are organized in the `.claude/` directory:

```
.claude/
├── CLAUDE.md              # Main instructions (project overview, quick start)
└── rules/
    ├── architecture.md    # Project structure, core components, design patterns
    ├── commands.md        # Available commands and supported resources
    ├── development.md     # Setup, common tasks, tech stack
    ├── code-style.md      # Python style, adding resources/commands
    ├── testing.md         # Test categories, fixtures, writing tests
    └── workflow.md        # Git branching, versioning, documentation
```

**See [.claude/CLAUDE.md](.claude/CLAUDE.md) for complete project instructions.**

## Quick Reference

### Project Overview

`dtctl` is a kubectl-like CLI tool for interacting with the Dynatrace REST API. The tool follows kubectl's design patterns:
- Resource-oriented commands (get, describe, create, delete, edit, apply, exec)
- Support for multiple output formats (JSON, YAML, table, CSV)
- Configuration management via config files
- Context switching between environments

**Important**: Use kubectl naming conventions (e.g., `exec` not `execute`). Exclude classic-environment v1 and v2 APIs.

### Current Version

**0.2.4** (defined in `pyproject.toml` and `src/dtctl/__init__.py`)

### Key Rules

1. **NEVER commit features directly to main** - Use feature branches
2. **ALL features MUST be documented BEFORE merging**
3. **ALL merges to main MUST increment the version number**
4. **ALL changes MUST be documented in CHANGELOG.md**

See [.claude/rules/workflow.md](.claude/rules/workflow.md) for complete workflow requirements.

### Common Commands

```bash
# Development
pip install -e ".[dev]"
ruff format src/
ruff check src/ --fix
pytest tests/ -v

# CLI
dtctl --help
dtctl get workflows
dtctl get dashboards --all  # Include ready-made documents
dtctl query "fetch logs | limit 10"
```

### Technology Stack

Python 3.10+, Typer, httpx, Pydantic v2, Rich, PyYAML, Jinja2

## Full Documentation

| Document | Description |
|----------|-------------|
| [.claude/CLAUDE.md](.claude/CLAUDE.md) | Main instructions and quick start |
| [.claude/rules/architecture.md](.claude/rules/architecture.md) | Project structure, design patterns |
| [.claude/rules/commands.md](.claude/rules/commands.md) | Commands and resources reference |
| [.claude/rules/development.md](.claude/rules/development.md) | Setup, tasks, tech stack |
| [.claude/rules/code-style.md](.claude/rules/code-style.md) | Code style guidelines |
| [.claude/rules/testing.md](.claude/rules/testing.md) | Testing conventions |
| [.claude/rules/workflow.md](.claude/rules/workflow.md) | Git workflow, versioning |
| [docs/IMPLEMENTATION_STATUS.md](docs/IMPLEMENTATION_STATUS.md) | Feature roadmap |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

---
> Source: [timstewart-dynatrace/Python-dtctl](https://github.com/timstewart-dynatrace/Python-dtctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
