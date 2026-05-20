## python-conventions

> Python Coding Conventions

# Python Coding Standards

## Style Guidelines
- Line length: 100 characters maximum
- Indentation: 4 spaces (no tabs)
- Quotes: Double quotes by default, avoid escaping when possible

## Naming Conventions
- Variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Classes: `PascalCase`
- Functions/Methods: `snake_case`
- Files: `snake_case.py`

## Import Organization
- Order: standard library → third-party → first-party
- Imports should be grouped and alphabetized
- Example:
  ```python
  import json
  import os
  
  import requests
  
  from src.core import utils
  ```

## Documentation
- Google-style docstrings required for:
  - Classes
  - Methods
  - Functions
- Example:
  ```python
  def function_name(param1: str, param2: int) -> bool:
      """One-line summary of function purpose.
      
      More detailed description if needed.
      
      Args:
          param1: Description of param1
          param2: Description of param2
          
      Returns:
          Description of return value
          
      Raises:
          ExceptionType: When and why this exception is raised
      """
  ```

## Type Annotations
- Type hints required for all code except tests
- Follow PEP 484 guidelines

---
> Source: [JustTryAI/databricks-mcp-server](https://github.com/JustTryAI/databricks-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
