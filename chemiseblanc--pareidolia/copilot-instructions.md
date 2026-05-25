## pareidolia

> This document outlines the development guidelines and best practices for AI agents working on the Pareidolia project.

# Agent Guidelines for Pareidolia Development

This document outlines the development guidelines and best practices for AI agents working on the Pareidolia project.

## Project Overview

Pareidolia is a Python CLI application for generating AI prompt template collections. It uses modular components (personas, tasks, examples) compiled via Jinja2 templates into complete prompt files.

## Working Memory

**All planning, requirements, design, and other working documents must be stored in the `.working-memory/` folder unless explicitly instructed otherwise.** This keeps project artifacts organized and separate from source code and user-facing documentation.

## Core Principles

### 1. Code Quality

- **Type Hints**: All functions, methods, and variables must use proper type hints
- **Linting**: Code must pass ruff and mypy checks before committing
- **Documentation**: Docstrings required for all public APIs (Google style)
- **Testing**: pytest test coverage for all new features

### 2. Architecture Guidelines

#### Module Organization

```
src/pareidolia/
├── __init__.py
├── __main__.py
├── cli.py                 # CLI entry point, argument parsing
├── core/
│   ├── __init__.py
│   ├── persona.py        # Persona data models and logic
│   ├── task.py           # Task data models and logic
│   ├── example.py        # Example data models and logic
│   └── library.py        # Library bundling logic
├── templates/
│   ├── __init__.py
│   ├── engine.py         # Jinja2 template engine
│   ├── loader.py         # Template loading and caching
│   └── defaults/         # Default template files
├── generators/
│   ├── __init__.py
│   ├── prompt.py         # Prompt generation
│   └── naming.py         # Tool-specific naming conventions
└── utils/
    ├── __init__.py
    ├── filesystem.py     # File I/O utilities
    └── validation.py     # Input validation
```

### 3. Design Patterns

- **Separation of Concerns**: Keep CLI, business logic, and I/O separate
- **Dependency Injection**: Pass dependencies rather than importing globals
- **Immutability**: Use dataclasses with frozen=True where appropriate
- **Type Safety**: Leverage mypy strict mode for maximum type checking

#### Example Type Hints

```python
from typing import Protocol, TypedDict
from pathlib import Path
from dataclasses import dataclass

class TemplateEngine(Protocol):
    def render(self, template: str, context: dict[str, Any]) -> str: ...

@dataclass(frozen=True)
class Persona:
    name: str
    description: str
    characteristics: list[str]
    
@dataclass(frozen=True)
class Task:
    name: str
    persona: str
    instructions: str
    output_format: str | None = None

def generate_prompt(
    persona: Persona,
    task: Task,
    examples: list[str],
    template_engine: TemplateEngine,
) -> str:
    """Generate a complete prompt from components.
    
    Args:
        persona: The persona definition
        task: The task definition
        examples: List of example outputs
        template_engine: Template rendering engine
        
    Returns:
        Rendered prompt content
        
    Raises:
        ValueError: If required fields are missing
    """
    ...
```

## Testing Requirements

### Test Structure

```
tests/
├── conftest.py              # Shared fixtures
├── unit/
│   ├── test_persona.py
│   ├── test_task.py
│   ├── test_template_engine.py
│   └── test_generators.py
├── integration/
│   ├── test_cli.py
│   └── test_end_to_end.py
└── fixtures/
    ├── personas/
    ├── tasks/
    └── templates/
```

### Testing Guidelines

- **Unit tests**: Test individual functions and classes in isolation
- **Integration tests**: Test CLI commands and workflows
- **Fixtures**: Use pytest fixtures for common test data
- **Parametrize**: Use `pytest.mark.parametrize` for multiple test cases
- **Coverage**: Aim for >90% code coverage
- **Test naming**: Use descriptive names: `test_<what>_<condition>_<expected>`

### Example Tests

```python
import pytest
from pareidolia.core.persona import Persona

def test_persona_creation_with_valid_data():
    persona = Persona(
        name="researcher",
        description="An expert researcher",
        characteristics=["thorough", "analytical"]
    )
    assert persona.name == "researcher"

@pytest.mark.parametrize("invalid_name", ["", " ", "invalid name", "123"])
def test_persona_creation_rejects_invalid_names(invalid_name: str):
    with pytest.raises(ValueError, match="Invalid persona name"):
        Persona(name=invalid_name, description="Test", characteristics=[])
```

## Development Workflow

1. **Start feature**:
   ```bash
   git checkout master
   git pull
   git checkout -b feature/library-creation
   ```

2. **Develop with tests**:
   - Write failing test
   - Implement feature
   - Ensure test passes
   - Run full test suite
   - Run linting

3. **Commit changes**:
   ```bash
   # Stage related changes
   git add src/pareidolia/core/library.py tests/unit/test_library.py
   
   # Commit with proper message
   git commit -m "core: add library bundling functionality

   Implement Library class to bundle multiple prompts with consistent
   naming conventions. Supports tool-specific prefixes for Claude Code
   and GitHub Copilot.
   
   - Add Library dataclass with validation
   - Implement prefix mapping for different tools
   - Add comprehensive unit tests"
   ```

4. **Verify before merge**:
   ```bash
   uv run pytest
   uv run ruff check .
   uv run mypy src/
   ```

5. **Merge to master**:
   ```bash
   git checkout master
   git merge --no-ff feature/library-creation
   git branch -d feature/library-creation
   ```

## Dependencies

- **Runtime**: Jinja2 (templating)
- **Development**: pytest, ruff, mypy, pytest-cov
- **Typing**: Use `typing` module, `typing_extensions` if needed

Add dependencies using uv:
```bash
uv add jinja2
uv add --dev pytest ruff mypy pytest-cov
```

## Common Tasks for AI Agents

When asked to implement features:

1. **Create feature branch** from master
2. **Read existing code** to understand patterns
3. **Add type hints** to all new code
4. **Write tests first** (TDD approach preferred)
5. **Implement feature** with proper error handling
6. **Run test suite** and ensure all tests pass
7. **Run linters** (ruff, mypy) and fix issues
8. **Commit with proper message** following guidelines
9. **Verify merge-readiness** before merging to master

## Error Handling

- Use specific exception types
- Provide helpful error messages
- Validate inputs early
- Use type hints to prevent errors

```python
class PareidoliaError(Exception):
    """Base exception for pareidolia."""

class PersonaNotFoundError(PareidoliaError):
    """Raised when a persona cannot be found."""

class InvalidTemplateError(PareidoliaError):
    """Raised when a template is invalid."""

def load_persona(name: str) -> Persona:
    """Load a persona by name.
    
    Args:
        name: Persona name to load
        
    Returns:
        Loaded persona
        
    Raises:
        PersonaNotFoundError: If persona does not exist
        ValueError: If name is empty or invalid
    """
    if not name or not name.strip():
        raise ValueError("Persona name cannot be empty")
    # ... implementation
```

## Extensibility Patterns

### Tool Adapter Pattern

Pareidolia uses the **Tool Adapter Pattern** for supporting different export formats (e.g., Copilot, Claude Code, standard). This pattern enables easy addition of new export types without modifying existing code.

#### Design

- **Base Class**: `ToolAdapter` (ABC with auto-registration)
- **Protocol**: `NamingConvention` (for type hints)
- **Registry**: Class-level dictionary for dynamic discovery
- **Auto-registration**: Via `__init_subclass__` hook

#### Adding a New Export Format

To add support for a new tool (e.g., "cursor"):

```python
from abc import ABC
from pathlib import Path
from pareidolia.generators.naming import ToolAdapter


class CursorNaming(ToolAdapter):
    """Cursor IDE naming convention."""

    @property
    def name(self) -> str:
        """Tool identifier used in config and CLI."""
        return "cursor"

    @property
    def description(self) -> str:
        """Human-readable description for help text."""
        return "Cursor IDE format (.cursor.md)"

    @property
    def file_extension(self) -> str:
        """File extension including the dot."""
        return ".cursor.md"

    def get_filename(self, action_name: str, library: str | None = None) -> str:
        """Generate filename for this tool's convention.
        
        Args:
            action_name: The action name
            library: Optional library name for bundling
            
        Returns:
            Generated filename
        """
        # Example: prefix with library if provided
        if library:
            return f"{library}_{action_name}.cursor.md"
        return f"{action_name}.cursor.md"

    def get_output_path(
        self,
        output_dir: Path,
        action_name: str,
        library: str | None = None,
    ) -> Path:
        """Generate full output path.
        
        Args:
            output_dir: Base output directory
            action_name: The action name
            library: Optional library name
            
        Returns:
            Complete output path
        """
        filename = self.get_filename(action_name, library)
        # Example: create subdirectory if library is specified
        if library:
            return output_dir / "cursor" / library / filename
        return output_dir / filename
```

**That's it!** The adapter is automatically registered when the class is defined. Users can now use `--tool cursor` in the CLI, and it will appear in the help text automatically.

#### Key Features

- **Auto-registration**: Subclasses register themselves on import
- **Validation**: Invalid tool names show helpful error with available options
- **Dynamic help**: CLI `--tool` help text lists all registered adapters
- **Type-safe**: Adapters satisfy `NamingConvention` protocol
- **Testable**: Registry can be cleared for test isolation

#### Testing New Adapters

```python
import pytest
from pathlib import Path
from pareidolia.generators.naming import ToolAdapter


def test_cursor_adapter_registered():
    """Verify cursor adapter is registered."""
    adapter = ToolAdapter.get_adapter("cursor")
    assert adapter.name == "cursor"


def test_cursor_naming_with_library():
    """Test cursor naming convention with library."""
    adapter = ToolAdapter.get_adapter("cursor")
    filename = adapter.get_filename("research", library="mylib")
    assert filename == "mylib_research.cursor.md"


def test_cursor_output_path():
    """Test cursor output path generation."""
    adapter = ToolAdapter.get_adapter("cursor")
    path = adapter.get_output_path(
        Path("/output"), "research", library="mylib"
    )
    assert path == Path("/output/cursor/mylib/mylib_research.cursor.md")
```

## Summary

- Write clean, typed, tested Python code
- Follow Linux kernel commit message format
- Use feature branches and logical commits
- Maintain high test coverage
- Run linters before committing
- Focus on modularity and separation of concerns
- Use established patterns (e.g., Tool Adapter) for extensibility

This ensures the Pareidolia codebase remains maintainable, well-tested, and easy to understand.

---
> Source: [Chemiseblanc/pareidolia](https://github.com/Chemiseblanc/pareidolia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
