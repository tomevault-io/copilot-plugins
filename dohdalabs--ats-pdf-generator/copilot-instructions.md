## ats-pdf-generator

> **ALWAYS run after editing Python code:**


# Python Standards (Python 3.13+)

## Quality Commands

**ALWAYS run after editing Python code:**

```bash
just format-python    # Auto-fix formatting
just quick-check      # Run all checks (lint, type, test, security)
```

## Required Patterns

### Imports (stdlib → third-party → local)

```python
# Standard library
from pathlib import Path
from typing import Any

# Third-party
import pytest

# Local
from ats_pdf_generator.utils import validate_data
```

### Functions (Type hints + docstrings required)

```python
def process_data(
    data: dict[str, Any],
    output_path: Path,
    *,
    validate: bool = True,
) -> bool:
    """Process data and return success status.

    Args:
        data: Input data dictionary
        output_path: Output file path
        validate: Whether to validate input

    Returns:
        True if successful, False otherwise

    Raises:
        ValidationError: If validation fails
    """
```

### Custom Exceptions

```python
class ATSGeneratorError(Exception):
    """Base exception for ATS PDF Generator."""

class ValidationError(ATSGeneratorError):
    """Input validation failed."""
```

### CLI Error Handling (CRITICAL for entry points)

```python
def cli() -> None:
    """CLI entrypoint with user-friendly error handling."""
    try:
        main()
    except ValidationError as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)
    except KeyboardInterrupt:
        print("\nOperation cancelled by user.", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Unexpected error: {e}", file=sys.stderr)
        sys.exit(2)

if __name__ == "__main__":
    cli()  # Use cli() not main()
```

### Tests (>90% coverage required)

```python
def test_function_behavior() -> None:
    """Test specific function behavior."""
    # Arrange
    input_data = {"key": "value"}

    # Act
    result = function_under_test(input_data)

    # Assert
    assert result == expected

def test_error_handling() -> None:
    """Test error conditions."""
    with pytest.raises(ValidationError):
        function_under_test(invalid_data)
```

## Checklist for AI Assistants

When editing Python code:

1. ✅ Add type hints to ALL functions (params + return)
2. ✅ Add docstrings to ALL public functions/classes
3. ✅ Use custom exceptions + CLI wrapper for errors
4. ✅ Organize imports (stdlib → third-party → local)
5. ✅ Write tests for new code (arrange-act-assert)
6. ✅ Run `just format-python` to auto-fix
7. ✅ Run `just quick-check` before completing
8. ✅ Fix ALL linter violations before finishing

## Key Rules

- **Type hints**: Use `dict[str, Any]` not `Dict[str, Any]` (Python 3.13+)
- **Error handling**: Chain exceptions with `from e`, use custom exception classes
- **CLI entry points**: MUST wrap main() with cli() for user-friendly errors
- **Tests**: >90% coverage, test both success and error paths
- **Validation**: Code MUST pass `just quick-check` before completion
- **Functions**: <50 lines, single responsibility, fail fast

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dohdalabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dohdalabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
