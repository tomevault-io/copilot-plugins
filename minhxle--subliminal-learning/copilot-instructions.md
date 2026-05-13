## subliminal-learning

> This document contains coding style and development guidelines for the subliminal learning project.

# Claude Development Guidelines

This document contains coding style and development guidelines for the subliminal learning project.

## Logging

**Use loguru instead of print statements for all logging.**

### Import and Basic Usage

```python
from loguru import logger

# Instead of print, use appropriate log levels:
logger.info("Starting process...")       # General information
logger.success("Process completed!")     # Success messages
logger.warning("This might be an issue") # Warnings
logger.error("Something went wrong")     # Errors
logger.exception("Full error details:")  # Errors with full traceback
logger.debug("Debug information")        # Debug details
```

### Log Levels

- `logger.info()` - General information about program flow
- `logger.success()` - Successful completion of operations
- `logger.warning()` - Potential issues that don't stop execution
- `logger.error()` - Errors that may cause failures
- `logger.exception()` - Errors with full traceback (use in except blocks)
- `logger.debug()` - Detailed information for debugging

### Examples

```python
# ❌ Don't use print
print(f"Processing {len(items)} items...")
print("Done!")

# ✅ Use loguru
logger.info(f"Processing {len(items)} items...")
logger.success("Processing completed successfully!")

# ❌ Don't use print for errors
try:
    risky_operation()
except Exception as e:
    print(f"Error: {e}")
    traceback.print_exc()

# ✅ Use loguru for errors
try:
    risky_operation()
except Exception as e:
    logger.error(f"Error in risky_operation: {e}")
    logger.exception("Full traceback:")
```

## Code Style

- Follow PEP 8 Python style guidelines
- Use type hints for function parameters and return values
- Use dataclasses with `kw_only=True` for configuration objects
- Keep functions focused on single responsibilities

## Testing

- Write tests for all new functionality
- Use pytest for test framework
- Include both unit tests and integration tests where appropriate

## Documentation

- Use clear, concise docstrings for all functions and classes
- Include type information in function signatures
- Document configuration options and their purposes

---
> Source: [MinhxLe/subliminal-learning](https://github.com/MinhxLe/subliminal-learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
