## playwright-failure-analyzer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Playwright Failure Analyzer is a GitHub Action that automatically analyzes Playwright test failures and creates comprehensive, well-formatted GitHub issues with optional AI-powered insights. It's built as a composite action using Python 3.11.

## Core Architecture

### Two-Phase Execution Model

The action runs in two distinct phases orchestrated by `action.yml`:

1. **Parse Phase** (`src/parse_report.py`):
   - Reads Playwright JSON report from configurable path (default: `playwright-report/results.json`)
   - Extracts failure information into structured `TestFailure` dataclasses
   - Limits failures to `max-failures` parameter (default: 3)
   - Writes intermediate JSON summary to `/tmp/failure_summary.json`
   - Sets GitHub Actions output: `failures-count`

2. **Issue Creation Phase** (`src/create_issue.py`):
   - Reads failure summary from temporary file
   - Optionally runs AI analysis via LiteLLM (supports OpenAI, Anthropic, OpenRouter, DeepSeek)
   - Formats comprehensive GitHub issue with markdown
   - Creates/updates GitHub issue via REST API
   - Sets outputs: `issue-number`, `issue-url`

### Key Components

- **Error Handling** (`src/error_handling.py`): Centralized error management with severity levels, error codes, and validation decorators. All errors use `ActionError` with suggestions for resolution.

- **AI Analysis** (`src/ai_analysis.py`): Optional LiteLLM integration that analyzes failure patterns, provides root cause analysis, and suggests fixes. Only runs if API keys are configured.

- **Utilities** (`src/utils.py`): Shared helpers for ANSI stripping, GitHub context parsing, duration formatting, and path manipulation.

## Development Commands

### Running Tests

```bash
# Run all tests with the test runner
python tests/run_tests.py

# Run specific test file
python -m unittest tests/test_parse_report.py

# Run with verbose output
python -m unittest discover -v tests/
```

### Code Quality

```bash
# Format code (Black)
black --line-length=100 src/ tests/

# Sort imports (isort)
isort --profile black --line-length 100 src/ tests/

# Lint code (Flake8)
flake8 src/ --max-line-length=100 --ignore=E501,W503

# Type checking (mypy)
mypy src/ --ignore-missing-imports --no-strict-optional

# Security scanning (Bandit)
bandit -r src/ -f screen
```

### Pre-commit Hooks

The repository uses extensive pre-commit hooks for security and quality:

```bash
# Install hooks
pre-commit install

# Run all hooks manually
pre-commit run --all-files

# Run specific hook
pre-commit run detect-secrets --all-files
```

**Security-first approach**: Secret scanning (detect-secrets, gitleaks) and security linting (Bandit) run before code quality checks.

## Testing Locally

### Simulating the Action Locally

```bash
# 1. Create a test Playwright report
cd /path/to/test-project
npx playwright test --reporter=json > test-results.json

# 2. Run parse phase
python src/parse_report.py \
  --report-path test-results.json \
  --max-failures 5 \
  --output-file /tmp/failure_summary.json

# 3. Set environment variables for GitHub context
export GITHUB_REPOSITORY="owner/repo"
export GITHUB_TOKEN="your-token"
export GITHUB_SHA="abc123"
export GITHUB_RUN_ID="123456"
export GITHUB_WORKFLOW="Test"
export GITHUB_ACTOR="username"
export GITHUB_SERVER_URL="https://github.com"

# 4. Run issue creation phase (dry-run without actual issue creation)
python src/create_issue.py \
  --summary-file /tmp/failure_summary.json \
  --issue-title "Test Failures" \
  --issue-labels "bug,test" \
  --deduplicate false \
  --ai-analysis false
```

### Testing AI Analysis

```bash
# Set AI provider credentials
export OPENROUTER_API_KEY="sk-or-v1-..."
export AI_MODEL="openrouter/deepseek/deepseek-chat"

# Run with AI enabled
python src/create_issue.py \
  --summary-file /tmp/failure_summary.json \
  --ai-analysis true \
  ...
```

## Important Patterns & Conventions

### Error Handling

Always use the error handling system for consistent user feedback:

```python
from error_handling import ActionError, ErrorCodes, ErrorSeverity

raise ActionError(
    code=ErrorCodes.FILE_NOT_FOUND,
    message=f"Report file not found: {path}",
    severity=ErrorSeverity.HIGH,
    suggestions=[
        "Ensure Playwright tests have run",
        "Check the report path configuration"
    ]
)
```

### Dataclass Usage

The codebase uses dataclasses extensively for structured data:

```python
@dataclass
class TestFailure:
    test_name: str
    file_path: str
    line_number: Optional[int]
    error_message: str
    stack_trace: str
    duration: float
    retry_count: int
```

### GitHub Actions Output

Set outputs using the new GitHub Actions format (not deprecated `::set-output::`):

```python
from utils import set_github_output

set_github_output("issue-number", "42")
```

### AI Analysis Integration

AI analysis is always optional and gracefully degrades if not available:

```python
try:
    from ai_analysis import analyze_failures_with_ai
    AI_ANALYSIS_AVAILABLE = True
except ImportError:
    AI_ANALYSIS_AVAILABLE = False
```

## File Structure

```
src/
├── parse_report.py      # Phase 1: Parse Playwright JSON report
├── create_issue.py      # Phase 2: Create GitHub issue
├── ai_analysis.py       # Optional AI analysis via LiteLLM
├── error_handling.py    # Centralized error management
└── utils.py            # Shared utility functions

tests/
├── run_tests.py        # Test runner with linting & type checking
├── test_parse_report.py
├── test_create_issue.py
├── test_ai_analysis.py
├── test_integration.py
└── test_utils.py

docs/                   # Comprehensive documentation
examples/               # Example workflow configurations
```

## Dependencies

**Production** (requirements.txt):
- `requests` - GitHub API interaction
- `litellm` - Multi-provider AI support (optional)
- `openai` - OpenAI API client (optional)

**Development** (pre-commit-config.yaml):
- black, isort, flake8, mypy - Code quality
- bandit - Security scanning
- detect-secrets, gitleaks - Secret detection
- markdownlint - Documentation linting

## Common Pitfalls

1. **CRITICAL - Bash Exit on Error**: GitHub Actions runs bash with `-e` flag by default, which causes the script to exit immediately when a command fails. This prevents the test exit code from being captured and the `test-failed` output from being set. **ALWAYS** use `set +e` before running tests and `set -e` after capturing the exit code. Without this, the analyzer step will be skipped even when tests fail.

2. **Path Handling**: Always use `get_relative_path()` from utils when displaying file paths in issues to avoid exposing system paths.

3. **ANSI Codes**: Strip ANSI escape codes from test output using `strip_ansi_codes()` before including in GitHub issues.

4. **GitHub API Rate Limits**: The `GitHubAPIClient` implements exponential backoff and retry logic. Don't bypass it.

5. **Deduplication**: When `deduplicate: true`, the action searches for existing open issues with the same title before creating new ones.

6. **Exit Codes**: `parse_report.py` exits with 0 even when failures are found - finding failures is expected behavior, not an error.

7. **Secrets**: Never log or expose GitHub tokens, API keys, or credentials. Use Bandit's `# nosec` annotations only for false positives.

## CI/CD Integration Notes

The action is designed to work with `continue-on-error: true` on the Playwright test step. Users should implement custom failure detection:

```yaml
- name: Run tests
  id: tests
  run: |
    set +e  # CRITICAL: Disable exit on error to capture test exit code
    npx playwright test
    TEST_EXIT_CODE=$?
    set -e  # Re-enable exit on error
    echo "test-failed=$([ $TEST_EXIT_CODE -ne 0 ] && echo 'true' || echo 'false')" >> $GITHUB_OUTPUT
    exit $TEST_EXIT_CODE
  continue-on-error: true

- name: Analyze failures
  if: steps.tests.outputs.test-failed == 'true'  # Not 'if: failure()'
  uses: decision-crafters/playwright-failure-analyzer@v1
```

**Why `set +e` is required**: GitHub Actions runs bash scripts with the `-e` flag (exit on error). When `npx playwright test` fails, the script exits immediately before setting the output variable. Using `set +e` temporarily disables this behavior, allowing the script to capture the exit code and set the output properly.

## Documentation

The `docs/` directory contains comprehensive guides:
- `HOW_IT_WORKS.md` - Architecture and workflow explanation
- `TESTING_INSTRUCTIONS.md` - Testing in your repository
- `TROUBLESHOOTING.md` - Common issues and solutions
- `AI_ASSISTANT_GUIDE.md` - Quick reference for AI assistants

Always keep documentation in sync with code changes.

---
> Source: [decision-crafters/playwright-failure-analyzer](https://github.com/decision-crafters/playwright-failure-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
