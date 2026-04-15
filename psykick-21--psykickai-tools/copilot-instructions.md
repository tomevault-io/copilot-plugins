## psykickai-tools

> You are an AI assistant helping develop a Python library following best practices. For all code generation and modifications, adhere to these guidelines:

# Python Library Development Guidelines

You are an AI assistant helping develop a Python library following best practices. For all code generation and modifications, adhere to these guidelines:

## Code Structure
- Organize code using a clear package structure:
  ```
  my_library/
  ├── my_library/
  │   ├── __init__.py
  │   ├── core/
  │   ├── utils/
  │   └── logging/
  ├── tests/
  ├── docs/
  ├── setup.py
  ├── pyproject.toml
  └── README.md
  ```
- Each module should have a single, well-defined responsibility
- Implement lazy loading for optional dependencies
- Use appropriate design patterns (Factory, Singleton, etc.) where beneficial

## Code Style
- Follow PEP 8 style guide strictly
- Use consistent 4-space indentation
- Maximum line length: 88 characters (Black formatter standard)
- Use meaningful variable and function names
- Include docstrings for all public modules, functions, classes, and methods
- Type hints are mandatory for all function parameters and return values

## Documentation
- Every module must have a module-level docstring explaining its purpose
- Function docstrings must follow Google style:
  ```python
  def function(param1: str, param2: int) -> bool:
      """Short description of function.
      
      Longer description if needed.
      
      Args:
          param1: Description of param1
          param2: Description of param2
          
      Returns:
          Description of return value
          
      Raises:
          ExceptionType: Description of when this exception occurs
      """
  ```
- Include usage examples in docstrings
- Maintain up-to-date README.md with installation and basic usage instructions

## Testing
- Write unit tests for all public functions using pytest
- Maintain minimum 90% code coverage
- Include both positive and negative test cases
- Mock external dependencies in tests
- Use fixtures and parametrize when appropriate
- Include integration tests for complex features

## Logging
- Use Python's built-in logging module
- Define clear log levels:
  - DEBUG: Detailed debugging information
  - INFO: General operational events
  - WARNING: Unexpected but handled events
  - ERROR: Serious issues that need attention
  - CRITICAL: System-level failures
- Include contextual information in log messages
- Allow configuration of log handlers and formatters

## Error Handling
- Use custom exception classes for library-specific errors
- Provide meaningful error messages
- Include traceback information where appropriate
- Handle edge cases gracefully
- Never suppress exceptions without good reason

## Performance
- Use generators for large datasets
- Implement caching where beneficial
- Profile code for bottlenecks
- Optimize critical paths
- Consider memory usage in data structures

## Security
- Sanitize all input data
- Use secure defaults
- Never expose sensitive information in logs
- Follow OWASP guidelines for any web-related functionality
- Use environment variables for configuration

## Package Management
- Define all dependencies in setup.py/pyproject.toml
- Specify minimum Python version
- Use semantic versioning
- Include all required package metadata
- Define optional dependencies appropriately

## Code Generation Rules
1. Always include proper imports at the top of files
2. Include type hints for all function parameters and return values
3. Add comprehensive docstrings
4. Implement error handling
5. Add logging statements at appropriate points
6. Include inline comments for complex logic
7. Generate corresponding test files

## Response Format
For each code generation task:
1. First analyze requirements
2. List dependencies needed
3. Generate code with proper structure
4. Include example usage
5. Add relevant tests
6. Provide setup instructions if needed

Always consider backward compatibility and future extensibility in all code changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/psykick-21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
