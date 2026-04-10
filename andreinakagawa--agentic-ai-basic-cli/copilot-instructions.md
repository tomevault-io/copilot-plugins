## agentic-ai-basic-cli

> **CRITICAL**: Only create or modify files when explicitly requested by the user. Do not proactively create documentation, examples, or helper files unless directly asked.

# Claude Code Instructions

## Core Principles

**CRITICAL**: Only create or modify files when explicitly requested by the user. Do not proactively create documentation, examples, or helper files unless directly asked.

## Project Context

This is a **generic, pluggable agentic AI CLI framework**. The goal is to provide a standalone, reusable CLI that makes it easy to create, test, and deploy agentic systems with any LLM provider or framework.

### Origin & Purpose

- **Origin**: Extracted from another project that had a specialized AI Agent with cli interface
- **Purpose**: Provide a clean, minimal chat CLI where developers can plug in any agent implementation
- **Target Users**: Developers building agentic systems who want a ready-to-use CLI interface
- **Design Philosophy**: Agent-agnostic, portable, microservice-ready architecture

### What This Project Is

- ✅ A generic chat CLI framework with session management
- ✅ A pluggable agent interface supporting any LLM provider/framework
- ✅ Message history management and conversation export
- ✅ A testing harness for agentic systems
- ✅ A foundation for rapid prototyping of AI agents

### What This Project Is NOT

- ❌ Domain-specific to data analysis
- ❌ Tied to any particular LLM provider
- ❌ A web application (CLI only)
- ❌ A production-ready service (it's a framework/starter)

### Architecture Principles

- **Agent Interface Over Provider Abstraction**: Define a generic agent contract, let implementations choose their provider/framework
- **Stateless Agents**: Agents receive context on each call, don't maintain conversation state
- **Dependency Injection**: All dependencies passed via constructors, no global state
- **CLI-Agent Boundary**: Strict separation - agents cannot import from CLI package
- **Portability First**: Agents must be extractable and usable in any context (web, notebook, batch, etc.)

## Development Environment

**Dependency Management**: This project uses `uv` for dependency management.

**IMPORTANT**: Always use `uv run` to execute commands in this repository.

Examples:
```bash
# Run tests
uv run pytest

# Run Python scripts/modules
uv run python script.py

# Run linting
uv run ruff check src tests

# Run formatting
uv run ruff format src tests

# Any other command
uv run <command>
```

## Working Guidelines

### File Operations
- **Only create or modify files when explicitly requested**
- Always prefer editing existing files over creating new ones
- Never proactively create documentation unless asked

### Task Management
- Use TodoWrite to plan and track multi-step tasks
- Break down complex features into smaller steps
- Mark todos as in_progress when starting, completed immediately when done
- Don't batch completions

### Architecture Focus
- Keep agents self-contained and microservice-ready
- Use dependency injection, avoid global state
- Design clear interfaces and contracts
- Ensure portability for future extraction as microservice
- **No domain-specific code**: Keep the framework generic and extensible
- **CLI layer is disposable**: Agents must work without the CLI (in web apps, notebooks, etc.)

## When Working on Tasks

1. **Understand requirements**: Ask clarifying questions if needed
2. **Plan**: Use TodoWrite for multi-step tasks
3. **Implement**: Write code following best practices
4. **Verify**: Run tests with `uv run pytest`
5. **Complete**: Mark todos as done immediately after verification

## Remember

- Always use `uv run` for all commands
- Only create files when explicitly requested
- Documentation goes in `docs/` folder when requested

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreinakagawa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andreinakagawa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
