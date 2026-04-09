## cuetiepy

> CueTiePy is a pure-Python implementation of the [CUE language](https://cuelang.org/). Unlike pycue which wraps the Go binary, this is a native Python implementation enabling CUE usage without external dependencies.

# CueTiePy Development Guide

## Project Overview

CueTiePy is a pure-Python implementation of the [CUE language](https://cuelang.org/). Unlike pycue which wraps the Go binary, this is a native Python implementation enabling CUE usage without external dependencies.

**Target:** Python 3.12+
**Parser:** Lark (EBNF-based)
**Spec:** https://cuelang.org/docs/reference/spec/

## Quick Start

```bash
# Install dependencies
uv sync --extra dev

# Run tests
uv run pytest

# Run linter
uv run ruff check src/ tests/

# Run formatter
uv run ruff format src/ tests/
```

## Architecture

```
src/cuetiepy/
├── api.py               # Public API: load(), loads(), validate(), export()
├── grammar/             # Lark grammar and parser
│   ├── cue.lark         # CUE grammar definition
│   └── parser.py        # Parser initialization
├── ast/                 # Abstract syntax tree
│   ├── nodes.py         # AST node dataclasses
│   └── transformer.py   # Lark parse tree → AST
├── values/              # Lattice value system
│   ├── base.py          # Value ABC
│   ├── atoms.py         # Null, Bool, Int, Float, String, Bytes + Types
│   ├── top_bottom.py    # Top (_) and Bottom (_|_)
│   ├── structs.py       # Struct with open/closed semantics
│   ├── lists.py         # List values
│   ├── disjunction.py   # Disjunction (|) handling
│   ├── bounds.py        # Numeric constraints
│   └── defaults.py      # Default value (*) handling
├── eval/                # Evaluation engine
│   ├── evaluator.py     # AST → Value transformation
│   ├── scope.py         # Lexical scope management
│   └── cycles.py        # Cycle detection
├── builtins/            # Built-in functions
│   └── core.py          # len, close, and, or, div, mod
├── interop/             # Python ↔ CUE conversion
│   ├── export.py        # CUE Value → Python dict
│   ├── import_.py       # Python dict → CUE Value
│   └── schema.py        # Validation utilities
├── errors/              # Error hierarchy
└── stdlib/              # Standard library (incremental)
```

## Key Concepts

### Lattice Value System

CUE values form a lattice. All values inherit from `Value` (in `values/base.py`):

```python
class Value(ABC):
    @abstractmethod
    def unify(self, other: "Value") -> "Value":
        """Greatest lower bound (& operator)"""
        ...

    @abstractmethod
    def subsumes(self, other: "Value") -> bool:
        """Check if self is more general than other"""
        ...

    @abstractmethod
    def to_python(self) -> Any:
        """Export to Python native type (only for concrete values)"""
        ...

    @property
    @abstractmethod
    def concreteness(self) -> Concreteness:
        """CONCRETE, INCOMPLETE, or BOTTOM"""
        ...
```

### Unification (`&`)

Unification computes the greatest lower bound:
- `_ & x = x` (Top is identity)
- `_|_ & x = _|_` (Bottom absorbs)
- `int & 42 = 42` (type constrains to concrete)
- `1 & 2 = _|_` (conflict produces bottom)

### Disjunction (`|`)

Disjunction computes the least upper bound:
- `1 | 2` creates a choice
- Normalized: subsumed values removed
- Unification distributes: `(a|b) & c = (a&c) | (b&c)`

### Types ARE Values

In CUE, types and values are unified:
- `int` is an incomplete value (IntType) that subsumes all integers
- `42` is a concrete value (Int) that only equals itself
- Both are `Value` instances with `unify()` and `subsumes()`

### Struct Semantics

- **Open** (default): Allows additional fields
- **Closed**: Rejects fields not in definition (`close()` or `#Definition`)
- **Field markers**: regular, optional (`?`), required (`!`)
- **Hidden fields**: Start with `_`, excluded from export
- **Definitions**: Start with `#`, recursively closed

### Concreteness

Values have three concreteness states:
- `CONCRETE`: Fully specified, can be exported to Python
- `INCOMPLETE`: Has constraints but not fully resolved (e.g., `int`, `>=5`)
- `BOTTOM`: Error/conflict state

## Common Development Tasks

### Adding a New Value Type

1. Create class in `values/` inheriting from `Value`
2. Implement `unify()`, `subsumes()`, `to_python()`, `concreteness`
3. Update evaluator to handle the type
4. Add tests in `tests/unit/values/`

### Adding a Built-in Function

1. Implement in `builtins/core.py`
2. Create `Builtin` instance with `name`, `impl`, `min_args`, `max_args`
3. Register in `BUILTINS` dict
4. Add tests

### Extending the Grammar

1. Edit `grammar/cue.lark`
2. Update `ast/transformer.py` to handle new productions
3. Update `eval/evaluator.py` to evaluate new AST nodes
4. Add parser tests and evaluation tests

## Testing

```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/unit/values/test_unification.py

# Run with coverage
uv run pytest --cov=cuetiepy

# Run tests matching pattern
uv run pytest -k "test_struct"

# Stop on first failure
uv run pytest -x

# Verbose output
uv run pytest -v
```

### Test Organization

- `tests/unit/` - Unit tests per module
- `tests/integration/` - End-to-end tests
- `tests/fixtures/` - CUE file fixtures

## CUE Language Reference

Key features from the spec:

1. **Unification (`&`)**: Greatest lower bound in lattice
2. **Disjunction (`|`)**: Least upper bound, normalized
3. **Top (`_`)**: Most general value, identity for `&`
4. **Bottom (`_|_`)**: Error/conflict, absorbing for `&`
5. **Bounds**: `>=2`, `<5`, `!=null` - constraints on values
6. **Defaults**: `*"tcp" | "udp"` - default for export
7. **Comprehensions**: `for x in list if cond { ... }`
8. **Cycles**: Reference cycles forbidden; structural cycles need escape

## Error Handling

All errors inherit from `CUEError`:
- `ParseError`: Syntax errors with source location
- `EvaluationError`: Runtime evaluation errors
- `ConcretenessError`: Attempt to export incomplete value
- `CycleError`: Reference or structural cycle detected
- `UnificationError`: Unification conflict

Errors carry source locations for good diagnostics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vputz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vputz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
