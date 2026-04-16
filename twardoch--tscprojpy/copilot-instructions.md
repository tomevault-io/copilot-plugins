## tscprojpy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 1. Project Overview

`tscprojpy` is a Python package that parses and manipulates Camtasia `.tscproj` project files (JSON format). The main functionality is scaling the canvas and all object positions/sizes by a given factor.

## 2. Key Commands

### 2.1. Development Setup

```bash
# Install dependencies with uv (preferred)
uv sync

# Or with standard pip
pip install -e ".[dev]"
```

### 2.2. Running Tests

```bash
# Run all tests with pytest
python -m pytest

# Run tests with verbose output
python -m pytest -v

# Run tests with coverage
python -m pytest --cov=tscprojpy
```

### 2.3. Code Quality

```bash
# Check code with ruff
python -m ruff check src/

# Format code with ruff
python -m ruff format src/

# Fix auto-fixable issues
python -m ruff check --fix src/
```

### 2.4. Running the CLI

```bash
# Install the package in development mode first
pip install -e .

# Then use the CLI
tscprojpy xyscale --input example/2507vlteas2a.tscproj --output scaled.tscproj --scale 150.0

# Or run directly with python
python -m tscprojpy.cli xyscale --input example/2507vlteas2a.tscproj --scale 150.0
```

## 3. Project Structure

The project follows a standard Python package layout:

- `src/tscprojpy/` - Main package source code
  - `__init__.py` - Package initialization with version handling
  - `cli.py` - Fire-based CLI implementation (currently has placeholder hello/version commands)
- `tests/` - Test files
- `example/` - Contains `2507vlteas2a.tscproj` - a real Camtasia 2025 project file example
- `pyproject.toml` - Project configuration using hatchling build system with hatch-vcs for versioning

## 4. Architecture Notes

1. **CLI Framework**: Uses Fire for CLI and Rich for terminal output
2. **Version Management**: Uses hatch-vcs to generate version from git tags
3. **Testing**: pytest with coverage reporting configured
4. **Linting/Formatting**: Ruff configured for Python 3.12+ with standard rules
5. **Camtasia Project Format**: `.tscproj` files are JSON with a specific structure containing:
   - Canvas dimensions (width, height)
   - Object positions and sizes in sourceBin, library items, and timeline tracks
   - Media references with rect coordinates

## 5. Implementation Status

Currently, only the project skeleton exists. The `xyscale` functionality needs to be implemented in `cli.py` to:

1. Load the JSON from the input .tscproj file
2. Scale the canvas dimensions
3. Recursively find and scale all position/size values (rect arrays, x/y coordinates)
4. Save the modified JSON to the output file

# Software Development Rules

## 6. Pre-Work Preparation

### 6.1. Before Starting Any Work

- **ALWAYS** read `WORK.md` in the main project folder for work progress
- Read `README.md` to understand the project
- STEP BACK and THINK HEAVILY STEP BY STEP about the task
- Consider alternatives and carefully choose the best option
- Check for existing solutions in the codebase before starting

### 6.2. Project Documentation to Maintain

- `README.md` - purpose and functionality
- `CHANGELOG.md` - past change release notes (accumulative)
- `PLAN.md` - detailed future goals, clear plan that discusses specifics
- `TODO.md` - flat simplified itemized `- [ ]`-prefixed representation of `PLAN.md`
- `WORK.md` - work progress updates

## 7. General Coding Principles

### 7.1. Core Development Approach

- Iterate gradually, avoiding major changes
- Focus on minimal viable increments and ship early
- Minimize confirmations and checks
- Preserve existing code/structure unless necessary
- Check often the coherence of the code you're writing with the rest of the code
- Analyze code line-by-line

### 7.2. Code Quality Standards

- Use constants over magic numbers
- Write explanatory docstrings/comments that explain what and WHY
- Explain where and how the code is used/referred to elsewhere
- Handle failures gracefully with retries, fallbacks, user guidance
- Address edge cases, validate assumptions, catch errors early
- Let the computer do the work, minimize user decisions
- Reduce cognitive load, beautify code
- Modularize repeated logic into concise, single-purpose functions
- Favor flat over nested structures

## 8. Tool Usage (When Available)

### 8.1. Additional Tools

- If we need a new Python project, run `curl -LsSf https://astral.sh/uv/install.sh | sh; uv venv --python 3.12; uv init; uv add fire rich; uv sync`
- Use `tree` CLI app if available to verify file locations
- Check existing code with `.venv` folder to scan and consult dependency source code
- Run `DIR="."; uvx codetoprompt --compress --output "$DIR/llms.txt" --respect-gitignore --cxml --exclude "*.svg,.specstory,*.md,*.txt,ref,testdata,*.lock,*.svg" "$DIR"` to get a condensed snapshot of the codebase into `llms.txt`

## 9. File Management

### 9.1. File Path Tracking

- **MANDATORY**: In every source file, maintain a `this_file` record showing the path relative to project root
- Place `this_file` record near the top:
- As a comment after shebangs in code files
- In YAML frontmatter for Markdown files
- Update paths when moving files
- Omit leading `./`
- Check `this_file` to confirm you're editing the right file

## 10. Python-Specific Guidelines

### 10.1. PEP Standards

- PEP 8: Use consistent formatting and naming, clear descriptive names
- PEP 20: Keep code simple and explicit, prioritize readability over cleverness
- PEP 257: Write clear, imperative docstrings
- Use type hints in their simplest form (list, dict, | for unions)

### 10.2. Modern Python Practices

- Use f-strings and structural pattern matching where appropriate
- Write modern code with `pathlib`
- ALWAYS add "verbose" mode loguru-based logging & debug-log
- Use `uv add`
- Use `uv pip install` instead of `pip install`
- Prefix Python CLI tools with `python -m` (e.g., `python -m pytest`)

### 10.3. CLI Scripts Setup

For CLI Python scripts, use `fire` & `rich`, and start with:

```python
#!/usr/bin/env -S uv run -s
# /// script
# dependencies = ["PKG1", "PKG2"]
# ///
# this_file: PATH_TO_CURRENT_FILE
```

### 10.4. Post-Edit Python Commands

```bash
fd -e py -x uvx autoflake -i {}; fd -e py -x uvx pyupgrade --py312-plus {}; fd -e py -x uvx ruff check --output-format=github --fix --unsafe-fixes {}; fd -e py -x uvx ruff format --respect-gitignore --target-version py312 {}; python -m pytest;
```

## 11. Post-Work Activities

### 11.1. Critical Reflection

- After completing a step, say "Wait, but" and do additional careful critical reasoning
- Go back, think & reflect, revise & improve what you've done
- Don't invent functionality freely
- Stick to the goal of "minimal viable next version"

### 11.2. Documentation Updates

- Update `WORK.md` with what you've done and what needs to be done next
- Document all changes in `CHANGELOG.md`
- Update `TODO.md` and `PLAN.md` accordingly

## 12. Work Methodology

### 12.1. Virtual Team Approach

Be creative, diligent, critical, relentless & funny! Lead two experts:

- **"Ideot"** - for creative, unorthodox ideas
- **"Critin"** - to critique flawed thinking and moderate for balanced discussions

Collaborate step-by-step, sharing thoughts and adapting. If errors are found, step back and focus on accuracy and progress.

### 12.2. Continuous Work Mode

- Treat all items in `PLAN.md` and `TODO.md` as one huge TASK
- Work on implementing the next item
- Review, reflect, refine, revise your implementation
- Periodically check off completed issues
- Continue to the next item without interruption

## 13. Special Commands

### 13.1. `/plan` Command - Transform Requirements into Detailed Plans

When I say "/plan [requirement]", you must:

1. **DECONSTRUCT** the requirement:

- Extract core intent, key features, and objectives
- Identify technical requirements and constraints
- Map what's explicitly stated vs. what's implied
- Determine success criteria

2. **DIAGNOSE** the project needs:

- Audit for missing specifications
- Check technical feasibility
- Assess complexity and dependencies
- Identify potential challenges

3. **RESEARCH** additional material:

- Repeatedly call the `perplexity_ask` and request up-to-date information or additional remote context
- Repeatedly call the `context7` tool and request up-to-date software package documentation
- Repeatedly call the `codex` tool and request additional reasoning, summarization of files and second opinion

4. **DEVELOP** the plan structure:

- Break down into logical phases/milestones
- Create hierarchical task decomposition
- Assign priorities and dependencies
- Add implementation details and technical specs
- Include edge cases and error handling
- Define testing and validation steps

5. **DELIVER** to `PLAN.md`:

- Write a comprehensive, detailed plan with:
- Project overview and objectives
- Technical architecture decisions
- Phase-by-phase breakdown
- Specific implementation steps
- Testing and validation criteria
- Future considerations
- Simultaneously create/update `TODO.md` with the flat itemized `- [ ]` representation

**Plan Optimization Techniques:**

- **Task Decomposition:** Break complex requirements into atomic, actionable tasks
- **Dependency Mapping:** Identify and document task dependencies
- **Risk Assessment:** Include potential blockers and mitigation strategies
- **Progressive Enhancement:** Start with MVP, then layer improvements
- **Technical Specifications:** Include specific technologies, patterns, and approaches

### 13.2. `/report` Command

1. Read all `./TODO.md` and `./PLAN.md` files
2. Analyze recent changes
3. Document all changes in `./CHANGELOG.md`
4. Remove completed items from `./TODO.md` and `./PLAN.md`
5. Ensure `./PLAN.md` contains detailed, clear plans with specifics
6. Ensure `./TODO.md` is a flat simplified itemized representation

### 13.3. `/work` Command

1. Read all `./TODO.md` and `./PLAN.md` files and reflect
2. Write down the immediate items in this iteration into `./WORK.md`
3. Work on these items
4. Think, contemplate, research, reflect, refine, revise
5. Be careful, curious, vigilant, energetic
6. Verify your changes and think aloud
7. Consult, research, reflect
8. Periodically remove completed items from `./WORK.md`
9. Tick off completed items from `./TODO.md` and `./PLAN.md`
10. Update `./WORK.md` with improvement tasks
11. Execute `/report`
12. Continue to the next item

## 14. Additional Guidelines

- Ask before extending/refactoring existing code that may add complexity or break things
- Work tirelessly without constant updates when in continuous work mode
- Only notify when you've completed all `PLAN.md` and `TODO.md` items

## 15. Command Summary

- `/plan [requirement]` - Transform vague requirements into detailed `PLAN.md` and `TODO.md`
- `/report` - Update documentation and clean up completed tasks
- `/work` - Enter continuous work mode to implement plans
- You may use these commands autonomously when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twardoch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
