## rouge

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.
## Purpose

This is an agent-execution guide, not a full project manual.
Keep this file concise, actionable, and focused on how agents should work in this repo.
For user-facing setup, command reference, and operational detail, see `README.md`.


## Repository Map (Current)

- `src/rouge/cli/` - Main Typer CLI (`rouge`) and command groups
- `src/rouge/adw/` - ADW CLI (`rouge-adw`) orchestration entrypoint
- `src/rouge/worker/` - Worker daemon (`rouge-worker`) and artifact/state handling
- `src/rouge/core/` - Shared models, database access, workflow orchestration, and Claude Code integration
- `src/rouge/core/workflow/` - Pipeline composition, step implementations, workflow registry
- `tests/` - Unit tests across CLI, worker, core, and workflow modules

## Workflow Routing Notes

- Workflow type resolution is centralized in `src/rouge/core/workflow/workflow_registry.py`.
- Use `get_pipeline_for_type(workflow_type)` as the public entrypoint for pipeline resolution.
- Built-in workflow types are `full`, `patch`, and `thin`.

## Execution Guardrails

- Use `uv`-based commands from `README.md` for setup and execution.
- After code changes, run targeted checks for modified areas; at minimum consider:
  - `uv run ruff check src/`
  - `uv run mypy`
  - `uv run pytest tests/ -v`
- Prefer targeted tests first, then broader suite runs as needed.

## Source of Truth

- **Coding rules and test conventions**: `CODING_STANDARDS.md`
- **Commands, environment variables, runtime behavior**: `README.md`
- **Artifact policy details**: `ARTIFACT_POLICY.md`

---
> Source: [bponghneng/rouge](https://github.com/bponghneng/rouge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
