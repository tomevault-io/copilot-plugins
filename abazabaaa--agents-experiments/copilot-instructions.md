## agents-experiments

> This is a Python development project that prioritizes code quality, maintainability, and reliable testing. Your role is to help develop robust, well-tested code that follows established patterns and best practices. The project uses modern Python tooling with uv as the package manager.

# Python Development Guidelines

<project_context>
This is a Python development project that prioritizes code quality, maintainability, and reliable testing. Your role is to help develop robust, well-tested code that follows established patterns and best practices. The project uses modern Python tooling with uv as the package manager.
</project_context>

## Core Development Principles

<python_execution>
**CRITICAL**: When executing Python code, the assistant must NEVER invoke Python directly. Instead, the assistant must ALWAYS use `uv run` to ensure proper environment isolation and dependency management.

<incorrect_approach>
**FORBIDDEN - Never use this:**
```bash
python - <<'PY'
import sys
print(f"Running on Python {sys.version}")
PY
```
</incorrect_approach>

<correct_approach>
**REQUIRED - Always use this:**
```bash
uv run python - <<'PY'
import sys
print(f"Running on Python {sys.version}")
PY
```
</correct_approach>
</python_execution>

## Development Guidelines

### Package Management with uv

<package_management>
<motivation>
We use uv exclusively for package management because it provides consistent, reproducible environments and faster dependency resolution than traditional tools.
</motivation>

<instructions>
1. **Install packages**: `uv add package`
2. **Run tools**: `uv run tool`
3. **Upgrade packages**: `uv add --dev package --upgrade-package package`
4. **Sync dependencies**: `uv sync` (run this frequently to ensure consistency)
5. **Always use uv commands** for any package operations
</instructions>
</package_management>

### Code Quality Standards

<code_quality>
<motivation>
High-quality code reduces bugs, improves maintainability, and makes collaboration easier. These standards help ensure our codebase remains professional and scalable.
</motivation>

<requirements>
- **Type hints**: Include for all function parameters and return values
- **Docstrings**: Write for all public APIs following Google style
- **Function design**: Keep functions focused and single-purpose
- **Consistency**: Follow established patterns in the existing codebase
- **Line limits**: Format code with 88-character line limits for readability
</requirements>

<formatting_guidelines>
When lines exceed 88 characters:
- **Strings**: Wrap using parentheses for implicit concatenation
- **Function calls**: Use multi-line format with proper indentation
- **Imports**: Split into multiple lines with one import per line
</formatting_guidelines>
</code_quality>

### Testing Excellence

<testing>
<motivation>
Comprehensive testing catches bugs early, documents expected behavior, and enables confident refactoring. Well-written tests serve as living documentation of how code should behave.
</motivation>

<testing_approach>
- **Framework**: Execute tests with `uv run pytest`
- **Coverage**: Write tests that cover edge cases and error conditions
- **New features**: Include tests for all new functionality
- **Bug fixes**: Add regression tests when fixing bugs
- **Migration note**: Tests in `tests_old/` need rewriting to modern standards
</testing_approach>
</testing>

### Version Control Best Practices

<version_control>
<commit_messages>
Write clear, descriptive commit messages that explain the change and its purpose.

**For user-reported issues**, acknowledge the reporter:
```bash
git commit --trailer "Reported-by: <user_name>"
```

**For GitHub issues**, reference the issue number:
```bash
git commit --trailer "Github-Issue: #<number>"
```

Focus commit messages on describing the problem solved and the approach taken.
</commit_messages>

<pull_requests>
- Write detailed PR descriptions focusing on:
  - The problem being solved
  - The high-level approach taken
  - Any important design decisions
- Add **Tom and Bob** as reviewers for all PRs
- Keep PR descriptions focused on the change itself, not the tools used
</pull_requests>
</version_control>

## Development Workflow

### Environment Setup

<setup>
Initialize your environment with:
```bash
uv lock && uv sync --group dev && uv run pre-commit install
```

This will:
1. Refresh the lockfile
2. Install the dev group into `.venv`
3. Register the hooks in the managed environment

**Note**: `uv run` automatically locks and syncs before each command. Pass `--locked`/`--frozen` in CI if you need deterministic lockfiles.
</setup>

### Code Formatting

<formatting>
Use **Ruff** for consistent Python formatting (scoped to `src/` and `tests/`):

<commands>
- **Format code**: `uv run ruff format src tests`
- **Check style**: `uv run ruff check src tests`
- **Apply fixes**: `uv run ruff check src tests --fix`
</commands>

<key_considerations>
- **Line length**: 88 characters maximum
- **Import sorting**: Follows isort conventions (I001)
- **Clean imports**: Remove unused imports for clean code
</key_considerations>
</formatting>

### Justfile Shortcuts

<justfile>
Prefer the project Justfile for common routines:
- `just format` / `just lint` / `just style` - Ruff formatting and checks
- `just typecheck` - Run `uv run ty check .`
- `just precommit` - Execute all pre-commit hooks (`uv run pre-commit run --all-files`)
- `just quality` - Chain formatting, linting, and type checking in one go
</justfile>

### Type Checking

<type_checking>
Ensure type safety with **ty**, Astral's blazingly fast type checker.

<command>
`uv run ty check`
</command>

<capabilities>
**What ty does:**
- Performs static type analysis on Python code
- Provides detailed, informative error messages
- Automatically discovers virtual environments and project structure
- Utilizes all CPU cores for exceptional performance
</capabilities>

<requirements>
**Requirements for your code:**
- Explicit None checks for Optional types
- Proper type narrowing for string operations
- All functions properly typed with annotations
</requirements>

**Note**: ty is pre-alpha but rapidly improving, created by the team behind Ruff and uv.
</type_checking>

### Pre-commit Hooks

<pre_commit>
The project uses pre-commit hooks defined in `.pre-commit-config.yaml`:

<hooks>
- **Prettier**: Formats YAML/JSON files
- **Ruff**: Enforces Python style
- **ty**: Validates type annotations
</hooks>

<updating_ruff>
To update Ruff in pre-commit:
1. Check latest version on PyPI
2. Update the `rev` in config
3. Commit the config update first
</updating_ruff>
</pre_commit>

## Problem Resolution Guide

### CI Failure Resolution

<ci_resolution>
When CI fails, address issues in this order for efficiency:

<resolution_steps>
1. **Run formatting tools first**: `uv run ruff format src tests`
2. **Fix type errors**: Identified by `uv run ty check src tests`
3. **Resolve linting issues**: Any remaining issues
</resolution_steps>

<ty_error_handling>
For type errors from ty:
- Examine the full line context in ty's detailed error messages
- Verify Optional type handling
- Add appropriate type narrowing
- Ensure function signatures match usage
- ty provides informative diagnostics explaining both what's wrong and how to fix it
</ty_error_handling>
</ci_resolution>

### Best Practices

<best_practices>
- **Check changes**: Run `git status` before committing to review changes
- **Format first**: Run formatters before type checkers to avoid cascading issues
- **Focused changes**: Make minimal, focused changes that address specific problems
- **Follow patterns**: Use existing code patterns for consistency
- **Document APIs**: Write clear docstrings for all public APIs
- **Test thoroughly**: Write comprehensive tests for new functionality
</best_practices>

## Trio/Trio-Asyncio Integration

<concurrency>
We run the pipeline inside Trio but call into asyncio-based SDKs via `trio_asyncio.open_loop()`. Keep these lessons in mind when touching concurrency code:

<guidelines>
- **Always wrap asyncio interactions** with `async with trio_asyncio.open_loop():` to ensure each batch gets a fresh loop
- **Disable HTTP keep-alives** or reinitialize transports between loops to avoid "Event loop is closed" errors when reusing clients
- **Use Trio primitives** (`Semaphore`, `move_on_after`, `open_memory_channel`) for concurrency and cancellation; never spawn bare `asyncio.create_task` from Trio code
- **Ensure checkpoints**: Long-running operations should yield checkpoints regularly so the nursery can shut down cleanly
- **Lightweight logging**: Logging/trace hooks should be lightweight; if heavy work is needed, offload with `trio.to_thread`
</guidelines>
</concurrency>

## Critical Behaviors

<critical_behaviors>
**MANDATORY WORKFLOW**:

1. **Always consult local documentation** before coding
   - Start at `docs/indexes/INDEX.md` for the complete map
   - Jump to relevant sub-index (e.g., OpenAI Agents SDK, Claude Code SDK, Trio/Trio‑Asyncio, FastMCP, Lark, UV, Ruff, Ty, Marimo)

2. **Use indexes to locate** authoritative references and examples
   - Verify exact APIs, options, and patterns in linked files

3. **Prefer demonstrated patterns** from examples and reference stubs over ad‑hoc implementations
</critical_behaviors>

### Index-Driven Workflow

<workflow>
<steps>
1. **Receive issue/feature** and sketch likely touch points in the codebase

2. **Navigate documentation**:
   - Open `docs/indexes/INDEX.md`
   - Pick matching sub-index (e.g., `openai-agents-sdk-index.md` or `claudecode-sdk-index.md`)
   - Follow its referenced paths

3. **Identify signatures and examples**:
   ```bash
   # List indexes
   ls docs/indexes
   
   # Search docs
   rg -n "<symbol or concept>" docs
   
   # Narrow scope
   rg -n "<ClassOrFn>" docs/<area>
   ```

4. **Plan the change** using verified APIs and example patterns
   - Note links to 2–3 canonical examples from the sub-index

5. **Implement with tests**
   - Cross-check the result against referenced docs before opening a PR
</steps>
</workflow>

---
> Source: [abazabaaa/agents_experiments](https://github.com/abazabaaa/agents_experiments) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
