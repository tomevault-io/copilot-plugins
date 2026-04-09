## linux-scripts

> Python development standards and best practices for production-ready code

# Python Development Standards

## Key Principles
- Write concise, technical responses with accurate Python examples
- Prioritize clarity, efficiency, and best practices in all workflows
- Use object-oriented programming for complex architectures and functional programming for data processing pipelines
- Implement proper resource utilization and optimization when applicable
- Use descriptive variable names that reflect the components they represent
- Follow PEP 8 style guidelines strictly

## Code Style & Formatting
- Follow PEP 8 strictly
- Use Black formatter with 88 character line length
- Apply isort for import organization
- Use type hints for all function parameters and return values
- snake_case for variables/functions, PascalCase for classes
- Avoid single-letter variables except in short loops
- Maximum 50 lines per function
- Use modern Python idioms (3.9+)
- Prefer f-strings over string formatting methods
- Use pathlib for file system operations

## Documentation Requirements
- Use Google-style docstrings for all public functions, classes, and methods
- Document all parameters, return values, and exceptions
- Include usage examples for complex functions
- Add module-level docstrings explaining purpose
- Include performance characteristics for critical functions

Example docstring format:
```python
def calculate_total(items: List[float], tax_rate: float = 0.1) -> float:
    """Calculate total price including tax.
    
    Args:
        items: List of item prices.
        tax_rate: Tax rate as decimal (default: 0.1 for 10%).
        
    Returns:
        Total price including tax.
        
    Raises:
        ValueError: If tax_rate is negative.
        
    """
```

## Project Structure
- Use standard Python package structure with `__init__.py`
- Organize code into logical modules and packages
- Separate concerns appropriately (business logic, data processing, presentation, persistence)
- Keep related functionality together
- Use configuration files (YAML/JSON) for application settings

## Modern Python Patterns
- Use dataclasses for simple data containers
- Implement enums for constants and options
- Use context managers for resource management
- Apply property decorators for computed attributes
- Use type annotations with Union, Optional, and generic types
- Prefer composition over inheritance
- Use async/await when beneficial for I/O operations

## Error Handling & Debugging
- Use specific exception types with descriptive messages
- Validate inputs early (fail-fast principle)
- Use context managers for resource management
- Don't suppress exceptions without proper logging
- Handle edge cases explicitly
- Use try-except blocks for error-prone operations (data loading, network requests)
- Implement proper logging for progress and errors
- Use debugging tools when necessary (pdb, logging)

## Performance & Optimization
- Use appropriate data structures (dict for lookups, set for membership)
- Prefer list comprehensions and generator expressions
- Consider memory usage for large datasets
- Profile code when performance is critical
- Implement proper resource utilization (CPU, memory)
- Use batch processing for large datasets when applicable
- Profile code to identify and optimize bottlenecks
- Consider parallel processing for CPU-intensive tasks

## Testing Standards
- Use pytest as testing framework
- Aim for >90% code coverage
- Test all public functions and methods
- Use descriptive test names that explain what's being tested
- Follow AAA pattern: Arrange, Act, Assert
- Mock external dependencies
- Include integration tests for complex systems
- Test edge cases and error conditions

## Import Organization
- Group imports: standard library, third-party, local
- Use absolute imports when possible
- No wildcard imports (`from module import *`)
- Use meaningful aliases for common modules
- Order imports alphabetically within groups

## Security & Best Practices
- Never commit secrets or API keys
- Use environment variables for configuration
- Validate and sanitize external inputs
- Use logging instead of print statements
- Keep dependencies updated
- Use virtual environments for project isolation
- Implement proper input validation for user-facing interfaces

## Data Processing Best Practices
- Implement efficient data loading patterns
- Use proper data validation and preprocessing pipelines
- Handle NaN/Inf values properly
- Use configuration management for different environments

## Common Libraries & Frameworks
- numpy (for numerical computations)
- pandas (for data manipulation)
- requests (for HTTP requests)
- tqdm (for progress bars)
- pyyaml (for configuration files)
- pathlib (for file system operations)
- click (for CLI applications)
- fastapi (for web APIs)
- pydantic 2.0 (for type safe)
- SQLModel (for database)

## Common Patterns to Follow
- Use dataclasses for simple data containers
- Implement `__str__` and `__repr__` for custom classes
- Use property decorators for computed attributes
- Apply context managers for resource management
- Create modular code structures with clear separation of concerns
- Use configuration files for application settings
- Implement proper logging and monitoring
- Use version control for tracking changes

## Anti-Patterns to Avoid
- Mutable default arguments
- Bare `except:` statements
- Global variables (use dependency injection)
- Magic numbers (define named constants)
- Deep nesting (use early returns)
- Code duplication
- Hardcoded paths and configurations
- Ignoring resource cleanup
- Poor error messages

## Project Setup Best Practices
1. Begin projects with clear problem definition and requirements analysis
2. Create modular code structures with separate files for different concerns
3. Use configuration files for application settings
4. Implement proper logging and monitoring
5. Use version control effectively
6. Set up proper dependency management (uv, requirements.txt, etc.)
7. Create comprehensive README with setup and usage instructions
8. Include example usage and demonstrations when applicable

## Code Review Guidelines
- Check for adherence to style guidelines
- Verify proper error handling
- Ensure adequate test coverage
- Review performance implications
- Check for security vulnerabilities
- Verify documentation completeness
- Ensure proper resource management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hminle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hminle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
