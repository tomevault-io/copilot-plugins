## david-s-new-project

> You are an AI coding assistant following our organization's coding standards. These rules MUST be followed for all code generation, refactoring, and suggestions.

# AI Coding Standards for Cursor IDE

You are an AI coding assistant following our organization's coding standards. These rules MUST be followed for all code generation, refactoring, and suggestions.

## Core Principles

1. **Readability First**: Code is read more often than written. Prioritize clarity over cleverness.
2. **DRY (Don't Repeat Yourself)**: Avoid code duplication. Extract common functionality.
3. **KISS (Keep It Simple, Stupid)**: Prefer simple, straightforward solutions.
4. **YAGNI (You Aren't Gonna Need It)**: Don't add functionality until it's actually needed.
5. **Single Responsibility**: Each function, class, or module should have one clear purpose.

## Function and Method Guidelines

- **Maximum function length**: 50 lines (preferred: 30 lines)
- **Maximum parameters**: 5 (preferred: 3)
- **Maximum cyclomatic complexity**: 10 (preferred: 5)
- **Maximum nesting depth**: 3 levels (preferred: 2)
- **Use dataclasses** for functions requiring more than 5 parameters

## Code Organization

Follow this module structure:
1. Module docstring
2. Standard library imports
3. Third-party imports
4. Local imports
5. Constants
6. Classes (public → private)
7. Functions (public → private)
8. `if __name__ == "__main__":` block

## Naming Conventions

- **Functions/Variables**: `snake_case`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private**: `_leading_underscore`
- **Boolean**: `is_`, `has_`, `can_` prefixes

## Documentation Requirements

- ✅ **ALL public functions** require Google-style docstrings
- ✅ **Type hints** required for ALL function parameters and returns
- ✅ **Module docstrings** for all modules
- ✅ **Class docstrings** for all classes
- ❌ **No obvious comments** - code should be self-documenting

## Type Hints

Use type hints for ALL function signatures:
```python
from typing import List, Dict, Optional, Any

def process_items(
    items: List[str],
    config: Optional[Dict] = None
) -> Dict[str, Any]:
    """Process items and return results."""
    pass
```

## Error Handling

- ✅ Catch **specific exceptions**, not bare `except:`
- ✅ **Fail fast** - validate inputs early
- ✅ **Provide context** in error messages
- ✅ Use **custom exceptions** for domain errors
- ❌ **Don't suppress** exceptions silently

## Testing Standards

- ✅ **80%+ coverage** target
- ✅ **Arrange-Act-Assert** pattern
- ✅ **Descriptive test names**: `test_function_scenario_expected_result()`
- ✅ **Mock external dependencies**
- ✅ Test **edge cases** and **error conditions**

## Code Quality Metrics

When generating code, ensure:
- Functions ≤ 50 lines
- Cyclomatic complexity ≤ 10
- Nesting depth ≤ 3 levels
- All public functions have docstrings
- Type hints present and correct
- Error handling appropriate
- No security vulnerabilities
- Tests included for new code

## Import Organization

```python
# 1. Standard library
import json
from datetime import datetime

# 2. Third-party
import boto3
from botocore.exceptions import ClientError

# 3. Local
from .config import load_config
```

## Performance Guidelines

- ✅ **Measure first** - profile before optimizing
- ✅ Use **appropriate data structures** (set for membership, dict for lookups)
- ✅ **Cache** expensive computations
- ✅ Use **async/await** for I/O-bound operations
- ❌ **Avoid premature optimization**

## When Generating Code

1. Always include docstrings for all public functions and classes
2. Use type hints consistently for all function signatures
3. Follow existing code patterns in the codebase
4. Document design decisions in comments when non-obvious
5. Keep functions focused and testable - single responsibility
6. Use meaningful variable names - avoid abbreviations unless standard
7. Include error handling for all external API calls
8. Add type hints even for internal functions

## Code Review Checklist

Before suggesting code, verify:
- [ ] Functions ≤ 50 lines, complexity ≤ 10
- [ ] All public functions have docstrings
- [ ] Type hints present and correct
- [ ] Error handling appropriate
- [ ] No security vulnerabilities
- [ ] Tests included for new code
- [ ] Follows naming conventions
- [ ] No code duplication
- [ ] Imports organized correctly

## References

For complete standards, see:
- `CODING_STANDARDS.md` - Full documentation
- `CODING_STANDARDS_QUICK_REF.md` - Quick reference
- `AI_PROMPT_STANDARDS.md` - Prompt writing guidelines

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dleigh-paa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
