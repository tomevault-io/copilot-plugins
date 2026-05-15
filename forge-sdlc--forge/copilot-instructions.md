## forge

> Forge is an AI-powered SDLC orchestrator that automates software development workflows using LangGraph, FastAPI, and Claude.

# Forge Development Guidelines

## Overview

Forge is an AI-powered SDLC orchestrator that automates software development workflows using LangGraph, FastAPI, and Claude.

## Tech Stack

- Python 3.11+ with LangGraph for workflow orchestration
- FastAPI for webhook handling
- Redis for event queuing and state checkpointing
- Anthropic Claude (via direct API or Vertex AI)
- Deep Agents for autonomous code implementation
- Podman for containerized code execution

## Project Structure

```
src/forge/
├── api/                 # FastAPI routes and middleware
├── integrations/        # Jira, GitHub, Agents, Langfuse clients
├── models/              # Domain models (workflow, events, etc.)
├── orchestrator/        # LangGraph workflow nodes and gates
├── prompts/v1/          # Versioned prompt templates
├── queue/               # Redis Streams producer/consumer
├── sandbox/             # Container runner for code execution
├── workspace/           # Git operations and workspace management
└── config.py            # Application configuration

containers/              # Container image and entrypoint
tests/                   # Unit and integration tests
```

## Commands

```bash
# Run tests
uv run pytest

# Run specific tests
uv run pytest tests/unit/ -v

# Linting
uv run ruff check src/

# Format code
uv run ruff format src/

# Type checking
uv run mypy src/forge/

# Start API server (dev)
uv run uvicorn forge.main:app --reload --port 8000 --host 0.0.0.0

# Start queue worker
uv run forge worker

# Build container
podman build -t forge-dev:latest containers/
```

## Code Style

- Use `X | None` instead of `Optional[X]` (PEP 604)
- Use `StrEnum` for string enums
- Use `contextlib.suppress()` instead of empty try/except
- Prefix unused parameters with `_`
- Keep functions focused and small

## Workflow Labels

| Label | Meaning |
|-------|---------|
| `forge:managed` | Ticket is managed by Forge |
| `forge:prd-pending` | Awaiting PRD approval |
| `forge:spec-pending` | Awaiting spec approval |
| `forge:plan-pending` | Awaiting epic plan approval |
| `forge:task-pending` | Awaiting task approval |
| `forge:blocked` | Workflow blocked, needs intervention |
| `forge:retry` | Trigger retry of failed step |

## GitHub PR Comment Commands

| Command | Where | Effect |
|---------|-------|--------|
| `/forge skip-gate <name>` | PR comment | Skip a named CI check (substring match); persists across pushes |
| `/forge unskip-gate <name>` | PR comment | Remove a previously set skip |

Commands are only active when the workflow is at a CI stage (`wait_for_ci_gate`, `ci_evaluator`, `attempt_ci_fix`).

## Container Execution

Tasks are implemented in ephemeral Podman containers:
- System prompt loaded from `src/forge/prompts/v1/container-system.md`
- Task file written to `.forge/task.json` (excluded from commits)
- Agent has full tool access via Deep Agents
- Changes committed locally, orchestrator handles push/PR

---
> Source: [forge-sdlc/forge](https://github.com/forge-sdlc/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
