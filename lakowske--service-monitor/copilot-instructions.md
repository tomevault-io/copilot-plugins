## service-monitor

> This project uses **uv** for dependency management and **virtual environments**. Always use the `.venv` virtual environment when working on this project.

# Service Monitor Project

## Development Environment

This project uses **uv** for dependency management and **virtual environments**. Always use the `.venv` virtual environment when working on this project.

## Code Quality & Standards

- **Ruff** - Fast, all-in-one Python linter and formatter (replaces Black, Flake8, isort, and Bandit)
  - Automatic code formatting with 120 character line length
  - Comprehensive linting with Google docstring conventions
  - Import sorting and security checks included
- **mdformat** - Markdown formatting with GitHub Flavored Markdown support
- **Pre-commit hooks** - Automated quality checks before every commit

## Testing & Coverage

- **Pytest** - Modern testing framework with proper project structure
- **Coverage reporting** - Minimum 80% code coverage required
- **HTML coverage reports** - Generated in `htmlcov/` directory
- **Integration testing** - Structured test organization

## Development Commands

- `pytest --cov=. --cov-report=term-missing --cov-fail-under=80 --cov-report=html` - Run tests with coverage
- `ruff format .` - Format code
- `ruff check .` - Run linting
- `ruff check --fix .` - Run linting with auto-fixes
- `pre-commit install` - Install pre-commit hooks
- `pre-commit run --all-files` - Run all pre-commit checks

## Logging Standards

All code should implement comprehensive logging with the following requirements:

- **Severity levels** - Use appropriate levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- **File location** - Include `__name__` or explicit file path in logger configuration
- **Line numbers** - Use `%(lineno)d` in formatter to capture line numbers
- **Operation context** - Log the operation being performed with descriptive messages
- **Variable tracking** - Include relevant variable names and their values in log messages

### Example Logging Configuration

```python
import logging

# Configure logger with comprehensive formatting
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - [%(filename)s:%(lineno)d] - %(message)s'
)
logger = logging.getLogger(__name__)

# Example usage
def process_data(data_id, data_content):
    logger.info(f"Starting data processing - data_id: {data_id}, content_length: {len(data_content)}")
    try:
        # Process data
        result = transform_data(data_content)
        logger.debug(f"Data transformation successful - result_type: {type(result)}, result_size: {len(result)}")
        return result
    except Exception as e:
        logger.error(f"Data processing failed - data_id: {data_id}, error: {str(e)}", exc_info=True)
        raise
```

## Quality Gates

Every commit must pass:

1. Code formatting and linting (Ruff)
1. Markdown formatting (mdformat)
1. All tests passing
1. Minimum 80% code coverage
1. No trailing whitespace
1. Proper file endings
1. Valid YAML syntax

## Important Instructions

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (\*.md) or README files. Only create documentation files if explicitly requested by the User

---
> Source: [lakowske/service-monitor](https://github.com/lakowske/service-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
