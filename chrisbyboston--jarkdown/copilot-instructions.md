## jarkdown

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Setup and Development
```bash
# Create virtual environment and install dependencies
python3 -m venv venv
source venv/bin/activate
pip install -e ".[dev]"

# Install pre-commit hooks (for development)
pre-commit install

# Run the tool (after pip install -e .)
jarkdown ISSUE-KEY
jarkdown ISSUE-KEY --output /path/to/output
jarkdown ISSUE-KEY --verbose
jarkdown --version

# Direct Python execution (if in venv)
python -m jarkdown.jarkdown ISSUE-KEY
```

### Testing
```bash
# Run all tests
pytest

# Run tests with coverage
pytest --cov=src/jarkdown --cov-report=term-missing

# Run specific test file
pytest tests/test_cli.py
pytest tests/test_components.py

# Run with verbose output
pytest -v

# Run a single test
pytest tests/test_components.py::TestJiraApiClient::test_successful_api_call
```

### Code Quality
```bash
# Run linter
ruff check .

# Fix linting issues
ruff check --fix .

# Format code
ruff format .

# Run pre-commit hooks manually
pre-commit run --all-files
```

### Building and Distribution
```bash
# Build package
python -m build

# Install locally for testing
pip install -e .
```

## Architecture

The codebase follows a modular architecture with clear separation of concerns:

### Core Components

1. **`src/jarkdown/jarkdown.py`** - Main entry point and CLI orchestrator
   - Contains `main()` function that handles CLI argument parsing
   - Contains `export_issue()` function that coordinates the workflow
   - All error handling and `sys.exit()` calls happen here, not in component classes

2. **`src/jarkdown/jira_api_client.py`** - JiraApiClient class
   - Manages all Jira REST API communication
   - Handles authentication and session management
   - Returns raw JSON data, no data transformation
   - Raises specific exceptions (AuthenticationError, IssueNotFoundError, JiraApiError)

3. **`src/jarkdown/attachment_handler.py`** - AttachmentHandler class
   - Downloads and saves attachments
   - Handles filename conflict resolution
   - Stream-based downloading for memory efficiency
   - Returns metadata about downloaded files

4. **`src/jarkdown/markdown_converter.py`** - MarkdownConverter class
   - Converts HTML to Markdown using markdownify
   - Replaces Jira attachment URLs with local file references
   - Composes final markdown structure with metadata
   - Handles various Jira URL patterns for attachments

5. **`src/jarkdown/exceptions.py`** - Custom exception hierarchy
   - `JarkdownError` - Base exception
   - `JiraApiError` - API communication errors
   - `AuthenticationError` - 401 errors
   - `IssueNotFoundError` - 404 errors
   - `AttachmentDownloadError` - Download failures
   - `ConfigurationError` - Config issues

### Workflow

1. CLI parses arguments and validates environment variables
2. Creates JiraApiClient with credentials
3. Fetches issue data from Jira API
4. AttachmentHandler downloads all attachments
5. MarkdownConverter creates markdown with local attachment links
6. Files are written to output directory

### Environment Configuration

Requires `.env` file with:
- `JIRA_DOMAIN`: Atlassian domain (e.g., company.atlassian.net)
- `JIRA_EMAIL`: User email for authentication
- `JIRA_API_TOKEN`: API token from Atlassian account settings

### Output Structure

Creates directory named after issue key:
```
ISSUE-KEY/
├── ISSUE-KEY.md     # Markdown file with issue content
└── attachments...   # Downloaded files with original names
```

### Testing Structure

- `tests/test_cli.py` - CLI integration tests (run in-process, no subprocess)
- `tests/test_components.py` - Unit tests for individual components
- `tests/data/` - JSON fixtures for mocked API responses

All dependencies managed in `pyproject.toml` (no requirements.txt).
- Never delete any file in the @docs/design/ directory.

---
> Source: [chrisbyboston/jarkdown](https://github.com/chrisbyboston/jarkdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
