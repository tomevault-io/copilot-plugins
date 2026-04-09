## django-dep-10-12-agent

> - Run agent: `uv --quiet run agent.py "Your question here"`

# Django DEP 10 & 12 Agent - Development Guide

## Commands
- Run agent: `uv --quiet run agent.py "Your question here"`
- Demo: `just demo`
- Lint: `just lint` (all files) or `just lint "file.py"` (specific file)
- Format justfile: `just fmt`
- Bootstrap environment: `just bootstrap`

## Code Style
- **Python version**: 3.12+ required, 3.13+ targeted
- **Line length**: 120 characters (per Black)
- **Linting**: Ruff with fixable rules enabled (A, B, C, D, E, F)
- **Imports**: Standard library first, third-party libs second, local imports last
- **Types**: Use Pydantic models for structured data and validation
- **Error handling**: Use explicit error handling with appropriate HTTP status codes
- **Naming**: Follow PEP 8 (snake_case for functions/variables, CamelCase for classes)
- **Package management**: Use uv for dependencies with script directive for script deps

## Project Structure
This agent uses pydantic-ai-slim for LLM interfacing with DEP 10 & 12 documents.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jefftriplett)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jefftriplett)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
