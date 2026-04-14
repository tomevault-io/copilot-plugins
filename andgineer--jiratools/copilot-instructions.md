## jiratools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Setup

Use the activation script to set up the development environment:
```bash
source ./activate.sh
```

**IMPORTANT**: Always activate the virtual environment before running any commands. Use `source ./activate.sh` before each command.

This script:
- Creates a virtual environment using UV (Astral's UV package manager)
- Installs dependencies from requirements.txt
- Installs the package in development mode

## Commands

### Core Application
```bash
source ./activate.sh && jiratools --help                    # Show main help
source ./activate.sh && jiratools clone --help              # Show clone command options
source ./activate.sh && jiratools transition --help         # Show transition command options
```

### Testing
```bash
source ./activate.sh && pytest -v --cov=src/ --cov-report=xml tests    # Run tests with coverage
source ./activate.sh && pytest tests/                                  # Run all tests
source ./activate.sh && pytest tests/test_clone_tickets.py             # Run specific test file
```

### Code Quality
```bash
source ./activate.sh && pre-commit run --all-files          # Run all pre-commit hooks
```

**IMPORTANT**: Always use `pre-commit run --all-files` for code quality checks. Never run ruff or mypy directly.

## Architecture

### Package Structure
- `src/jiratools/` - Main package
  - `main.py` - CLI entry point using rich-click
  - `config.py` - JIRA connection configuration
  - `clone_tickets.py` - Ticket cloning functionality
  - `transition_tickets.py` - Ticket transition functionality

### Authentication
JIRA authentication uses environment variables:
- `JIRA_USERNAME` - JIRA username
- `JIRA_PASSWORD` - JIRA password

Default server: `https://jira.btmd.ru`

### CLI Structure
The application uses rich-click for the CLI with two main commands:
- `clone` - Clone JIRA tickets with customizable JQL queries
- `transition` - Transition tickets based on project and transition name

### Testing Framework
- Uses pytest with unittest.mock for mocking JIRA API calls
- Test fixtures in `tests/conftest.py` provide mock JIRA instances
- Coverage reports generated in XML format for CI integration

### Code Quality Tools
- Ruff for linting and formatting (line length: 100 chars for src, 99 for tests)
- MyPy for type checking (excludes tests directory)
- Pre-commit hooks enforce code quality standards
- Comprehensive ruff rules including security, complexity, and style checks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andgineer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andgineer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
