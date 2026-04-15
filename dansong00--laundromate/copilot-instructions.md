## laundromate

> Handles user authentication, profile data, and access control.

# Python Docstring Standards

Always update docstrings to reflect the current context of the file. Be concise and only present relevant details.

## Requirements:
- **Functions/Methods**: Include purpose, parameters, return values, and exceptions
- **Classes**: Describe purpose and key attributes
- **Modules**: Explain the module's role in the system
- **Keep it concise**: Only include information that's actually relevant
- **Update when code changes**: Docstrings must match current implementation

## Examples:

### Function Docstring:
```python
def create_access_token(subject: str, expires_minutes: Optional[int] = None) -> str:
    """Create a JWT access token for user authentication.

    Args:
        subject: User identifier (usually email)
        expires_minutes: Token expiration time (defaults to settings)

    Returns:
        Encoded JWT token string
    """
```

### Class Docstring:
```python
class User(Base):
    """User model representing authenticated users in the system.

    Handles user authentication, profile data, and access control.
    """
```

### Module Docstring:
```python
"""Authentication and security utilities for the LaundroMate API.

Provides JWT token management, password hashing, and user authentication.
"""
```

## When Editing:
- If you change function parameters, update the docstring
- If you modify class behavior, update the class docstring
- If you add new functionality, document it immediately
- Remove outdated information that's no longer relevant
---
alwaysApply: true
globs: *.py
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dansong00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dansong00)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
