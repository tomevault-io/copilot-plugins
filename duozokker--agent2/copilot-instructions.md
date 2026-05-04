## agent2

> This is Agent2, a framework for turning domain experts into production AI agents. Built on PydanticAI and FastAPI.

# Agent2 Copilot Instructions

This is Agent2, a framework for turning domain experts into production AI agents. Built on PydanticAI and FastAPI.

## When writing agent code:
- Import create_agent from shared.runtime
- Import create_app from shared.api
- Define output schemas as Pydantic BaseModel with Field() descriptions
- Use @agent.tool_plain for tool registration
- Use instructions= parameter (not system_prompt=)

## When writing framework code (shared/):
- Always use from __future__ import annotations
- Errors must be RFC 7807 via ProblemError
- Config must use frozen dataclasses
- Type hints on all function signatures

## File structure for agents:
agents/<name>/
  __init__.py
  schemas.py    - Pydantic output model
  agent.py      - create_agent() + tool registration
  tools.py      - Domain logic functions
  config.yaml   - Agent configuration
  main.py       - create_app() entrypoint
  Dockerfile    - Docker build instructions

---
> Source: [duozokker/agent2](https://github.com/duozokker/agent2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
