## type-bridge

> **type-bridge** is a Python ORM (Object-Relational Mapper) for TypeDB, designed to provide Pythonic abstractions over TypeDB's native TypeQL query language.

# Project Overview

**type-bridge** is a Python ORM (Object-Relational Mapper) for TypeDB, designed to provide Pythonic abstractions over TypeDB's native TypeQL query language.

TypeDB is a strongly-typed database with a unique type system that includes:

- **Entities**: Independent objects with attributes
- **Relations**: Connections between entities with role players
- **Attributes**: Values owned by entities and relations

## Python Version

This project requires **Python 3.13+** (see .python-version)

## Quick Start

```bash
# Install dependencies
uv sync --extra dev

# Install pre-commit hooks (required for development)
pre-commit install

# Run tests
uv run pytest                    # Unit tests (fast, no deps)
./test-integration.sh            # Integration tests (with Docker)

Podman users: integration tests work with Podman too—set `CONTAINER_TOOL=podman` (or `podman-compose`) so the fixtures call `podman compose ...` instead of Docker. Either runtime works as long as a compatible *compose* subcommand is available and the TypeDB container can start.

# Run examples
uv run python examples/basic/crud_01_define.py

# Code quality (run before committing)
uv run ruff check --fix .        # Lint and auto-fix
uv run ruff format .             # Format code
uvx ty check .                   # Type check library
uv run pyright tests/            # Type check tests
```

## Project Structure

```text
type_bridge/
├── __init__.py           # Main package exports
├── query.py              # TypeQL query builder
├── session.py            # Database connection and transaction management
├── reserved_words.py     # Reserved words validation
├── typedb_driver.py      # TypeDB driver wrapper
├── validation.py         # Validation utilities
├── attribute/            # Modular attribute system
│   ├── base.py           # Abstract Attribute base class
│   ├── string.py         # String attribute
│   ├── integer.py        # Integer attribute
│   ├── double.py         # Double attribute
│   ├── boolean.py        # Boolean attribute
│   ├── datetime.py       # DateTime attribute
│   ├── datetimetz.py     # DateTimeTZ attribute
│   ├── date.py           # Date attribute
│   ├── decimal.py        # Decimal attribute
│   ├── duration.py       # Duration attribute
│   └── flags.py          # Flag system (Key, Unique, Card, TypeFlags)
├── models/               # Base Entity and Relation classes (modularized)
│   ├── __init__.py       # Module exports
│   ├── base.py           # Base model functionality
│   ├── entity.py         # Entity class
│   ├── relation.py       # Relation class
│   ├── role.py           # Role definitions
│   └── utils.py          # Model utilities
├── expressions/          # Query expression system
│   ├── __init__.py
│   ├── aggregate.py      # Aggregation expressions
│   ├── base.py           # Base expression classes
│   ├── boolean.py        # Boolean expressions
│   ├── comparison.py     # Comparison expressions
│   ├── functions.py      # Function expressions
│   ├── role_player.py    # Role player expressions
│   └── string.py         # String expressions
├── fields/               # Field system
│   ├── __init__.py
│   ├── base.py           # Base field classes
│   └── role.py           # Role field definitions
├── crud/                 # CRUD operations (modularized)
│   ├── __init__.py       # Module exports (backward compatible)
│   ├── base.py           # Type variables (E, R)
│   ├── utils.py          # Shared utilities (format_value, is_multi_value_attribute)
│   ├── exceptions.py     # CRUD exceptions
│   ├── hooks.py          # Lifecycle hooks (CrudEvent, HookCancelled, CrudHook, HookRunner)
│   ├── entity/           # Entity CRUD operations
│   │   ├── __init__.py   # Entity module exports
│   │   ├── manager.py    # EntityManager class
│   │   ├── query.py      # EntityQuery chainable queries
│   │   └── group_by.py   # GroupByQuery for aggregations
│   └── relation/         # Relation CRUD operations
│       ├── __init__.py   # Relation module exports
│       ├── manager.py    # RelationManager class
│       ├── query.py      # RelationQuery chainable queries
│       ├── group_by.py   # RelationGroupByQuery for aggregations
│       └── lookup.py     # Relation lookup functionality
├── schema/               # Modular schema management
│   ├── __init__.py       # Module exports
│   ├── manager.py        # SchemaManager for schema operations
│   ├── info.py           # SchemaInfo container
│   ├── diff.py           # SchemaDiff for comparison
│   ├── migration.py      # MigrationManager for migrations
│   └── exceptions.py     # SchemaConflictError for conflict detection
└── generator/            # Code generator (TQL → Python)
    ├── __init__.py       # Public API: generate_models(), parse_tql_schema()
    ├── __main__.py       # CLI: python -m type_bridge.generator
    ├── models.py         # ParsedSchema, AttributeSpec, EntitySpec, RelationSpec
    ├── parser.py         # TypeQL parser with inheritance resolution
    ├── naming.py         # kebab-case → PascalCase/snake_case utilities
    ├── annotations.py    # Type annotations handling
    └── render/           # Code renderers
        ├── __init__.py   # Render module exports
        ├── api_dto.py    # Pydantic API DTO generation
        ├── attributes.py # Attribute class generation
        ├── entities.py   # Entity class generation
        ├── relations.py  # Relation class generation
        ├── package.py    # Package __init__.py generation
        ├── functions.py  # Function rendering
        └── registry.py   # Registry rendering

examples/
├── basic/                # Basic CRUD examples (start here!)
│   ├── crud.py               # Combined CRUD example
│   ├── crud_01_define.py     # Schema definition
│   ├── crud_02_insert.py     # Data insertion
│   ├── crud_03_read.py       # Fetching API
│   ├── crud_04_update.py     # Update operations
│   ├── crud_05_filter.py     # Advanced filtering
│   ├── crud_06_aggregate.py  # Aggregations
│   ├── crud_07_delete.py     # Delete operations
│   └── crud_08_put.py        # Idempotent PUT
├── advanced/             # Advanced features
│   ├── schema_01_manager.py       # Schema operations
│   ├── schema_02_comparison.py    # Schema comparison
│   ├── schema_03_conflict.py      # Conflict detection
│   ├── features_01_pydantic.py    # Pydantic integration
│   ├── features_02_type_safety.py # Literal types
│   ├── features_03_string_repr.py # String representations
│   ├── features_04_base_flag.py   # Python-only base classes
│   ├── features_05_implicit_flags.py # Implicit TypeFlags
│   ├── query_01_expressions.py    # Query expressions
│   ├── crud_07_chainable_operations.py # Chainable CRUD operations
│   ├── config_01_typename_case.py     # TypeName case configuration
│   ├── config_02_attribute_case.py    # Attribute case configuration
│   ├── config_03_typeflags.py         # TypeFlags configuration
│   ├── validation_01_reserved_words.py # Keyword validation
│   └── validation_02_edge_cases.py    # Validation edge cases
└── patterns/             # Design patterns
    ├── cardinality_01_multi_value.py  # Cardinality patterns
    └── inheritance_01_abstract.py     # Abstract types

tests/
├── unit/                 # Unit tests (fast, isolated, no external dependencies)
│   ├── core/             # Core functionality tests
│   ├── attributes/       # Attribute type tests
│   ├── flags/            # Flag system tests
│   ├── fields/           # Field system tests
│   ├── expressions/      # Query expression API
│   ├── crud/             # CRUD unit tests (lookup parser, etc.)
│   ├── query/            # Query builder tests
│   ├── session/          # Session tests
│   ├── exceptions/       # Exception tests
│   ├── generator/        # Code generator unit tests
│   ├── validation/       # Reserved word and keyword validation
│   └── type-check-except/ # Type check exception tests
└── integration/          # Integration tests (require running TypeDB)
    ├── crud/             # CRUD operations
    ├── queries/          # Query builder tests
    ├── schema/           # Schema operations
    ├── session/          # TransactionContext tests
    ├── validation/       # Validation integration tests
    └── generator/        # Generator integration tests (generate + import)
```

## Dependencies

The project requires:

- `typedb-driver>=3.7.0`: Official Python driver for TypeDB connectivity
- `pydantic>=2.12.4`: For validation and type coercion
- `isodate>=0.7.2`: ISO 8601 date/time parsing
- `lark>=1.1.9`: Parser toolkit for TypeQL schema parsing
- `jinja2>=3.1.0`: Template engine for code generation
- `typer>=0.15.0`: CLI framework for generator and migration tools
- Uses Python's built-in type hints and dataclass-like patterns

## Documentation

Full documentation site: **[https://ds1sqe.github.io/type-bridge/](https://ds1sqe.github.io/type-bridge/)**

### User Documentation

- **[README.md](README.md)** - Quick start guide for users

### Development Guides

- **[docs/development/setup.md](docs/development/setup.md)** - Development setup, commands, code quality standards
- **[docs/development/testing.md](docs/development/testing.md)** - Testing strategy, patterns, and execution

### TypeDB Integration

- **[docs/development/typedb.md](docs/development/typedb.md)** - TypeDB concepts, driver API, TypeQL syntax, 3.x changes
- **[docs/development/abstract-types.md](docs/development/abstract-types.md)** - Abstract types and interface hierarchies in TypeDB

### Architecture & Internals

- **[docs/development/internals.md](docs/development/internals.md)** - Internal type system, ModelAttrInfo, modern Python standards

### User Guide (API docs)

- **[docs/guide/index.md](docs/guide/index.md)** - API overview and quick reference
- **[docs/guide/attributes.md](docs/guide/attributes.md)** - Attribute types and value types
- **[docs/guide/entities.md](docs/guide/entities.md)** - Entity definition and ownership
- **[docs/guide/relations.md](docs/guide/relations.md)** - Relations, roles, and role players
- **[docs/guide/abstract-types.md](docs/guide/abstract-types.md)** - Abstract types implementation and patterns
- **[docs/guide/cardinality.md](docs/guide/cardinality.md)** - Card API and Flag system
- **[docs/guide/crud.md](docs/guide/crud.md)** - CRUD operations and managers
- **[docs/guide/queries.md](docs/guide/queries.md)** - Query expressions and aggregations
- **[docs/guide/schema.md](docs/guide/schema.md)** - Schema management and conflict detection
- **[docs/guide/generator.md](docs/guide/generator.md)** - Code generator (TQL → Python)
- **[docs/guide/validation.md](docs/guide/validation.md)** - Pydantic integration and type safety

## Getting Help

- `/help`: Get help with using Claude Code
- Report issues at: <https://github.com/anthropics/claude-code/issues>

## Quick Reference

### Basic Usage Pattern

```python
from type_bridge import Entity, TypeFlags, AttributeFlags, String, Integer, Flag, Key

# 1. Define attribute types
class Name(String):
    pass

# Override attribute type name (for legacy schemas)
class EmailAddress(String):
    flags = AttributeFlags(name="email")  # TypeDB: attribute email, value string;

class Age(Integer):
    pass

# 2. Define entity with ownership
class Person(Entity):
    flags = TypeFlags(name="person")
    name: Name = Flag(Key)
    email: EmailAddress
    age: Age | None = None  # Optional field

# 3. Create instances (keyword arguments required)
alice = Person(name=Name("Alice"), email=EmailAddress("alice@example.com"), age=Age(30))

# 4. CRUD operations
person_manager = Person.manager(db)
person_manager.insert(alice)
persons = person_manager.all()
```

### Key Concepts

1. **Attributes are independent types** - Define once, reuse across entities/relations
2. **Use TypeFlags for entities/relations** - Clean API with `TypeFlags(name="person")`
3. **Use AttributeFlags for attributes** - Override names with `AttributeFlags(name="person_name")` or use case formatting with `AttributeFlags(case=TypeNameCase.SNAKE_CASE)`
4. **Use Flag system** - `Flag(Key)`, `Flag(Unique)`, `Flag(Card(min=2))`
5. **Python inheritance maps to TypeDB supertypes** - Use `abstract=True` for abstract types
6. **Keyword-only arguments** - All Entity/Relation constructors require keyword arguments
7. **TransactionContext** - Share transactions across operations with `db.transaction()`
8. **Connection type** - Managers accept `Database`, `Transaction`, or `TransactionContext`
9. **Dict helpers** - Use `to_dict()` and `from_dict()` for serialization

## Claude Code Instructions

### Always Fix Issues

**CRITICAL**: When encountering bugs, test failures, or issues during development:

- ALWAYS fix issues, regardless of whether they appear to be "pre-existing"
- Never dismiss bugs as "not related to current changes" - if you found it, fix it
- Complete the work fully; don't leave partial implementations
- If a refactor is done, it should be COMPLETE - no dead code, no overridden methods that defeat the purpose

### Code Quality Standards

**CRITICAL: Always run checks on the ENTIRE codebase, not just modified files.**

AI sessions may include multiple separate conversations or context compactions, so changes from earlier in a session could introduce issues. Running checks on only a subset of files will miss these problems and cause CI failures.

Before committing, run ALL of these on the entire codebase:

```bash
uv run ruff check --fix .        # Lint entire codebase
uv run ruff format .             # Format entire codebase
uv run pyright type_bridge/      # Type check library
uv run pyright tests/            # Type check tests
uv run pytest tests/unit/ -x     # Run all unit tests
```

Before PRs, also run:

```bash
./test-integration.sh            # Integration tests (requires TypeDB)
```

Additional standards:

- When unifying code (e.g., EntityManager + RelationManager), remove ALL duplicated methods from subclasses
- Delete dead code completely - no backwards-compatibility hacks, no `_unused` variables, no `# removed` comments

### Pydantic Best Practices

When working with Pydantic model validators:

- **Avoid `object.__setattr__`** - Using `object.__setattr__` to bypass Pydantic's validation is a hack that can mask bugs
- **Use `mode='before'` validators** to transform input data BEFORE validation, avoiding recursion with `validate_assignment=True`
- **Use `mode='wrap'` validators** only when you need to capture state before AND after validation (e.g., preserving private attributes)
- **Access private attributes via `__pydantic_private__`** - This is Pydantic's official API for private attribute storage

Example pattern for value transformation:

```python
@model_validator(mode="before")
@classmethod
def transform_input(cls, values: Any) -> dict[str, Any]:
    """Transform input BEFORE validation - no recursion possible."""
    if isinstance(values, dict):
        data = dict(values)
        # Transform values here
        return data
    return values
```

For detailed documentation, see the links above.

---
> Source: [ds1sqe/type-bridge](https://github.com/ds1sqe/type-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
