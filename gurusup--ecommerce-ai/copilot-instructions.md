## ecommerce-ai

> Rules for backend domain entities in hexagonal architecture


# Backend Domain Entities Rules

## Purpose
Domain entities represent the core business objects and encapsulate business logic. They are the heart of the hexagonal architecture's domain layer.

## Structure Guidelines

### Entity Definition
- Use `@dataclass` decorator for entity classes
- Place validation logic in `__post_init__` method
- Include business methods that operate on the entity
- Keep entities framework-agnostic (no external dependencies)

### Validation Pattern
```python
@dataclass
class EntityName:
    """Entity description."""
    # Fields with defaults
    id: Optional[str] = None
    field: str = ""
    
    def __post_init__(self):
        """Validate the entity."""
        if not self.field or not self.field.strip():
            raise ValueError("Field cannot be empty")
```

### Business Methods
- Add methods that represent business operations
- Validate inputs before modifying state
- Raise meaningful domain exceptions for business rule violations

### Examples
Reference existing entities:
- [User entity](mdc:backend/src/domain/entities/user.py)
- [Product entity](mdc:backend/src/domain/entities/product.py)
- [Order entity](mdc:backend/src/domain/entities/order.py)

## Best Practices
1. Keep entities pure - no external framework dependencies
2. Validate data in `__post_init__` and business methods
3. Use meaningful field names and types
4. Include docstrings for all entities and methods
5. Raise domain-specific exceptions for validation failures

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gurusup)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gurusup)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
