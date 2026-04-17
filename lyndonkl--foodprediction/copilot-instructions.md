## foodprediction

> - **Personal project** for food prediction using metabolomics data

# Cursor Rules for Food Prediction ML Project

## Project Context
- **Personal project** for food prediction using metabolomics data
- **Senior engineer** with 13 years full-stack + 3 years ML experience
- **Python-focused** with emphasis on clean, modular architecture
- **No testing requirements** - focus on working code over test coverage

## Code Quality Standards

### Architecture Principles
- **Modular design**: Separate concerns into focused modules
- **Minimal coupling**: Keep dependencies between modules loose
- **Clear interfaces**: Well-defined function signatures and data contracts
- **Single responsibility**: Each function/class has one clear purpose
- **Functional over OOP**: Prefer pure functions and data structures over complex class hierarchies

### Code Style
- **Clean and readable**: Self-documenting code with clear variable names
- **Minimal boilerplate**: Avoid unnecessary abstractions or over-engineering
- **Early returns**: Use guard clauses to reduce nesting
- **Type hints**: Use TypeScript-style type annotations for clarity
- **Consistent formatting**: Follow PEP 8 with 4-space indentation

### Data Handling
- **Immutable data**: Prefer immutable data structures where possible
- **Clear data flow**: Make data transformations explicit and traceable
- **Efficient processing**: Use vectorized operations (pandas/numpy) over loops
- **Memory conscious**: Avoid unnecessary data copying or large intermediate objects

## Development Workflow

### Implementation Approach
1. **Break down problems** into small, manageable steps
2. **Brainstorm solutions** before coding - discuss trade-offs
3. **Start simple** - implement basic functionality first
4. **Iterate and refine** - add complexity only when needed
5. **Keep it working** - prefer working code over perfect code

### File Organization
- **Logical grouping**: Group related functionality in modules
- **Clear naming**: Use descriptive file and function names
- **Minimal nesting**: Keep directory structure shallow and intuitive
- **Index files**: Use `__init__.py` to expose clean APIs

### Code Structure
```python
# Preferred function structure
def process_data(data: pd.DataFrame) -> Dict[str, Any]:
    """Clear docstring explaining purpose and return value."""
    # Early validation
    if data.empty:
        return {}
    
    # Main processing logic
    result = transform_data(data)
    
    # Return clean result
    return result
```

## ML-Specific Guidelines

### Data Processing
- **Pipeline approach**: Chain data transformations clearly
- **Validation**: Check data quality at each step
- **Caching**: Cache expensive computations (embeddings, etc.)
- **Memory efficiency**: Process data in chunks when needed

### Model Development
- **Simple first**: Start with baseline models before complex ones
- **Clear metrics**: Define evaluation criteria upfront
- **Reproducible**: Use random seeds and version data
- **Incremental**: Build models step by step

### Performance
- **Profile first**: Measure before optimizing
- **Vectorized operations**: Use numpy/pandas efficiently
- **Lazy evaluation**: Process data on-demand when possible
- **Memory management**: Be conscious of large datasets

## Communication Style

### When Working Together
- **Ask clarifying questions** before implementing
- **Suggest alternatives** when appropriate
- **Explain trade-offs** for design decisions
- **Break down complex tasks** into steps
- **Focus on practical solutions** over academic perfection

### Code Reviews
- **Highlight potential issues** constructively
- **Suggest improvements** with reasoning
- **Respect existing patterns** unless clearly problematic
- **Keep feedback actionable** and specific

## Project-Specific Rules

### File Naming
- Use snake_case for Python files: `data_processor.py`
- Use descriptive names: `dreams_embeddings.py` not `embeddings.py`
- Group related files in directories: `src/data/`, `src/models/`

### Documentation
- **Clear docstrings** for public functions
- **Type hints** for function signatures
- **README updates** for major changes
- **Inline comments** only for complex logic

### Error Handling
- **Fail fast**: Catch errors early with clear messages
- **Graceful degradation**: Handle missing data appropriately
- **Logging**: Use appropriate log levels (info, warning, error)
- **No silent failures**: Make problems visible

## Anti-Patterns to Avoid
- **Over-engineering**: Don't add complexity without clear benefit
- **Premature optimization**: Profile before optimizing
- **Deep inheritance**: Prefer composition over inheritance
- **Magic numbers**: Use named constants
- **Long functions**: Break down functions > 20 lines
- **Nested conditionals**: Use early returns and guard clauses

## Success Criteria
- **Working code** that solves the problem
- **Readable and maintainable** structure
- **Efficient processing** of metabolomics data
- **Clear data flow** from raw data to predictions
- **Modular architecture** that's easy to extend

Remember: This is a personal project. Focus on getting things working efficiently rather than perfect engineering practices. Keep it simple, clean, and functional. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lyndonkl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
