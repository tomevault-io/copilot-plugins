## testing

> pattern: "**/*test*.py"

type: auto-attached
pattern: "**/*test*.py"

---

# Testing Guidelines

## Test Structure
- Unit tests in `tests/unit/`
- Integration tests in `tests/integration/`
- Use pytest fixtures for setup
- Mark tests: `@pytest.mark.unit` or `@pytest.mark.integration`

## Coverage Requirements
- Minimum 80% coverage
- Run with: `task test`
- View report: `open htmlcov/index.html`

## Test Patterns
```python
import pytest
from devopsdaysdc25.module import function

def test_function_success():
    """Test successful case."""
    result = function(valid_input)
    assert result == expected

def test_function_error():
    """Test error handling."""
    with pytest.raises(SpecificError):
        function(invalid_input)

@pytest.fixture
def sample_data():
    """Provide test data."""
    return {"key": "value"}
```

---
> Source: [JonZeolla/devopsdaysdc25](https://github.com/JonZeolla/devopsdaysdc25) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
