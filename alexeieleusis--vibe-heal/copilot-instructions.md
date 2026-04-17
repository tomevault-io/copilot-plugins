## vibe-heal

> This document provides a comprehensive overview of the `vibe-heal` project, designed to assist developers and contributors in understanding its architecture, development conventions, and operational procedures.

# Vibe-Heal Project Overview

This document provides a comprehensive overview of the `vibe-heal` project, designed to assist developers and contributors in understanding its architecture, development conventions, and operational procedures.

## 1. Project Purpose and Core Functionality

`vibe-heal` is a Python-based command-line tool that automates the remediation of code quality issues identified by SonarQube. It leverages AI coding assistants, such as Claude Code and Aider, to generate and apply fixes.

### Key Features

- **Automated Issue Remediation**: Fetches issues from a SonarQube server and uses AI to fix them.
- **AI Tool Integration**: Supports both Claude Code and Aider for code generation.
- **Git Integration**: Automatically commits each successful fix with a conventional commit message.
- **Branch Cleanup**: Can analyze and fix all modified files in a given Git branch.
- **Code Deduplication**: Provides functionality to identify and refactor duplicated code blocks.
- **Configuration**: Flexible configuration options via environment files.

## 2. Technical Stack and Architecture

- **Language**: Python 3.11+
- **CLI Framework**: `typer`
- **Configuration**: `pydantic-settings` for loading configuration from environment files.
- **HTTP Client**: `httpx` for asynchronous communication with the SonarQube API.
- **AI Integration**: The system uses a factory pattern (`AIToolFactory`) to instantiate the appropriate AI tool (Claude Code or Aider).
- **Orchestration**: The core logic is managed by orchestrator classes (`VibeHealOrchestrator`, `CleanupOrchestrator`, `DeduplicationOrchestrator`) that coordinate the workflow of fetching issues, invoking the AI tool, and committing changes.

## 3. Detailed Architecture and Module Responsibilities

The high-level architecture of `vibe-heal` is as follows:

```txt
CLI Layer (cli.py)
    ↓
Orchestrator (orchestrator.py) - coordinates entire workflow
    ↓
├─ Config (config/) - loads .env.vibeheal
├─ SonarQubeClient (sonarqube/) - fetches issues via API
├─ IssueProcessor (processor/) - sorts/filters issues
├─ AITool (ai_tools/) - fixes issues (ClaudeCodeTool implemented)
└─ GitManager (git/) - creates commits
```

### Module Responsibilities

- **`config/`**: Handles Pydantic-based configuration from `.env.vibeheal` or `.env` files. It supports token or basic authentication, AI tool auto-detection, and custom environment files.
- **`sonarqube/`**: Contains the async HTTP client for the SonarQube Web API. It can fetch issues, manage projects, and run analyses. The `ProjectManager` class handles the lifecycle of temporary SonarQube projects for branch cleanup.
- **`processor/`**: Implements the business logic for handling issues, including sorting them by line number in descending order and filtering by fixability and severity.
- **`ai_tools/`**: Defines the abstract interface for AI tools and their implementations. It includes a factory for creating AI tool instances and supports both Claude Code and Aider.
- **`git/`**: Manages Git operations using GitPython. It creates conventional commits for each fix and includes a `BranchAnalyzer` to identify modified files in a branch.
- **`orchestrator.py`**: Coordinates the main workflow for fixing files. It validates preconditions, fetches issues, processes them, and orchestrates the AI-powered fixing process.
- **`cleanup/`**: Manages the branch cleanup workflow. The `CleanupOrchestrator` analyzes a branch, creates a temporary SonarQube project, and iteratively fixes issues in modified files.
- **`deduplication/`**: Contains the logic for removing code duplications. It includes a client for the SonarQube Duplications API, a processor for sorting and filtering duplications, and orchestrators for single-file and branch-wide deduplication.
- **`cli.py`**: Implements the command-line interface using `typer`, providing commands for fixing files, cleaning up branches, managing configuration, and more.

## 4. Critical Implementation Details

- **Reverse Line Order**: Issues and duplications are fixed from the highest line number to the lowest to prevent line number shifts from invalidating subsequent fixes.
- **Component Path Construction**: SonarQube API queries use the format `components=projectkey:filepath` to ensure accuracy.
- **Issue Status Filtering**: The tool uses `issueStatuses=OPEN,CONFIRMED` for precise issue filtering.
- **Git Safety**: Before fixing a file, the tool checks for uncommitted changes in that specific file to prevent data loss.
- **Asynchronous Operations**: `asyncio` and `await` are used for non-blocking communication with the SonarQube API and AI tools.
- **Snippet-Based Prompts for Deduplication**: To improve token efficiency, prompts for code duplication include only the first and last three lines of large duplicated blocks.

## 5. Development and Contribution Guide

### 5.1. Building and Running the Project

**Installation:**

To set up the development environment, use the following command. This will create a virtual environment using `uv` and install the necessary dependencies and pre-commit hooks.

```bash
make install
```

**Running Tests:**

Execute the test suite using `make`. This command runs `pytest` and generates a coverage report.

```bash
make test
```

**Linting and Type-Checking:**

To ensure code quality and adherence to style guides, run the following command. This will execute `pre-commit` hooks, `mypy` for type-checking, and `deptry` for dependency analysis.

```bash
make check
```

**Building the Project:**

To create a wheel file for distribution, use the build command:

```bash
make build
```

### 5.2. Running the Application

The main entry point for the application is the `vibe-heal` command.

**Common Commands:**

- **Fix a file**:

  ```bash
  vibe-heal fix path/to/your/file.py
  ```

- **Clean up a branch**:

  ```bash
  vibe-heal cleanup --base-branch origin/main
  ```

- **Remove code duplications**:

  ```bash
  vibe-heal dedupe path/to/your/file.py
  ```

- **View configuration**:

  ```bash
  vibe-heal config
  ```

### 5.3. Development Workflow

**Running `vibe-heal` locally:**

1. Install the project in development mode:

   ```bash
   uv pip install -e .
   ```

2. Create a `.env.vibeheal` file with your SonarQube configuration.
3. Test the configuration:

   ```bash
   vibe-heal config
   ```

4. Run a dry-run to preview changes:

   ```bash
   vibe-heal fix src/file.py --dry-run
   ```

5. Fix a single issue to test the end-to-end flow:

   ```bash
   vibe-heal fix src/file.py --max-issues 1
   ```

**Testing specific modules:**

- To run a specific test file:

  ```bash
  uv run pytest tests/sonarqube/test_client.py -v
  ```

- To run a single test function:

  ```bash
  uv run pytest tests/sonarqube/test_client.py::TestSonarQubeClient::test_get_issues_for_file -v
  ```

### 5.4. Testing Philosophy

- **Coverage**: All code should be covered by tests. `pytest-cov` is used to generate coverage reports.
- **Test Discovery**: `pytest` is configured to discover tests in the `tests/` directory.
- **Fixtures**: Test fixtures for SonarQube API responses are located in `tests/sonarqube/fixtures/api_responses.json`.

### 5.5. Development Conventions

- **Commits**: The project follows the Conventional Commits specification. Each fix is committed with a message that includes the SonarQube rule that was violated.
- **Code Style**: Code style is enforced using `ruff` and `pre-commit` hooks.
- **Typing**: The project uses type hints, and `mypy` is used to enforce static type checking.
- **Asynchronous Code**: The application uses `asyncio` for non-blocking I/O, particularly for interacting with the SonarQube API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexeieleusis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
