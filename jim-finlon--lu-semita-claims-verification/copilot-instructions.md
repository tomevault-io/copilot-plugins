## lu-semita-claims-verification

> - Follow PEP 8 style guide

# Cursor Rules for Lu Semita Claims Validation Project

## Code Quality Standards

### Python Style
- Follow PEP 8 style guide
- Use type hints for all function signatures
- Maximum line length: 100 characters
- Use descriptive variable names (avoid abbreviations except common ones like `i`, `n`, `x`)
- Document all public functions and classes with docstrings (Google style)

### Code Organization
- One class per file for major components
- Group related functions in modules
- Use `__init__.py` files to expose public API
- Keep functions focused and single-purpose (SRP)

### Testing Standards
- All test functions must start with `test_`
- Use descriptive test names: `test_requirement_id_description`
- Each test should test one specific requirement
- Include both positive and negative test cases where applicable
- Use assertions with clear error messages

### Error Handling
- Use specific exception types, not bare `except:`
- Include context in error messages
- Log errors appropriately (use logging module)
- Fail fast with clear error messages

### Performance
- Use NumPy/PyTorch vectorized operations when possible
- Avoid unnecessary loops in numerical code
- Cache expensive computations (e.g., prime lists)
- Profile before optimizing

### Documentation
- All modules must have module-level docstrings
- All classes must have class-level docstrings
- All public functions must have docstrings with Args, Returns, Raises sections
- Include examples in docstrings for complex functions
- Keep comments focused on "why", not "what"

### Dependencies
- Pin dependency versions in requirements.txt
- Avoid unnecessary dependencies
- Document why each dependency is needed
- Use virtual environments

### Type Safety
- Use type hints for function parameters and return values
- Use `typing` module for complex types (List, Dict, Tuple, Optional, Union)
- Use `from __future__ import annotations` for forward references

### Numerical Code
- Use appropriate precision (float64 for most calculations)
- Handle edge cases (division by zero, log(0), etc.)
- Validate inputs (check ranges, types, shapes)
- Use relative tolerances for floating-point comparisons
- Document numerical assumptions

### Git Practices
- Write clear commit messages
- Commit related changes together
- Use meaningful branch names

### Code Review Checklist
- [ ] Type hints present
- [ ] Docstrings complete
- [ ] Tests pass
- [ ] No linter errors
- [ ] Edge cases handled
- [ ] Error messages clear
- [ ] Performance acceptable
- [ ] Dependencies justified

## Project-Specific Rules

### Test Framework
- All tests inherit from `LuSemitaTestBase`
- Use `assertMetric()` for threshold checks
- Record results using `recordResult()`
- Follow SPEC-001 procedures exactly

### Algorithm Implementations
- Match consolidated summary code exactly
- Preserve mathematical formulas as comments
- Include version numbers in docstrings
- Document any deviations from source

### Measurement Functions
- All measurement functions in `test_framework/measurements.py`
- Functions must be pure (no side effects)
- Include input validation
- Return structured results (dicts or dataclasses)

### Data Handling
- Use DataCollector for all data storage
- Save results in JSON format for portability
- Include timestamps and metadata
- Version test data files

## File Naming Conventions
- Test files: `test_*.py`
- Implementation files: `*_impl.py` or descriptive names
- Configuration files: `*.yaml` or `*.json`
- Documentation: `*.md`

## Import Organization
1. Standard library imports
2. Third-party imports
3. Local application imports
4. Separate groups with blank lines

## Example Good Code

```python
from typing import List, Tuple
import numpy as np
import torch

def compute_mae(predictions: np.ndarray, 
                ground_truth: np.ndarray) -> float:
    """
    Compute Mean Absolute Error.
    
    Args:
        predictions: Array of predicted values, shape (N,)
        ground_truth: Array of true values, shape (N,)
    
    Returns:
        MAE value as float
    
    Raises:
        ValueError: If arrays have different lengths
    
    Example:
        >>> pred = np.array([1.0, 2.0, 3.0])
        >>> true = np.array([1.1, 1.9, 3.1])
        >>> compute_mae(pred, true)
        0.06666666666666667
    """
    if len(predictions) != len(ground_truth):
        raise ValueError(
            f"Arrays must have same length: "
            f"{len(predictions)} != {len(ground_truth)}"
        )
    return np.mean(np.abs(predictions - ground_truth))
```

## Anti-Patterns to Avoid
- ❌ Magic numbers (use named constants)
- ❌ Deeply nested conditionals (extract functions)
- ❌ Copy-paste code (extract common functionality)
- ❌ Silent failures (raise exceptions)
- ❌ Hardcoded paths (use Path objects and config)
- ❌ Unused imports (remove them)
- ❌ Commented-out code (delete it, use git history)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jim-finlon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
