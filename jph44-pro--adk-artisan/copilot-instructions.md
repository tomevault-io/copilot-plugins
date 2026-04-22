## adk-artisan

> Python code should have explicit type annotations for all variables, function parameters, and return types. This prevents mypy errors and improves code clarity and maintainability.


# Python Type Annotations Required

## Context
Python code should have explicit type annotations for all variables, function parameters, and return types. This prevents mypy errors and improves code clarity and maintainability.

Missing type annotations lead to mypy errors like:
- `Need type annotation for "variable_name" (hint: "variable_name: list[<type>] = ...")  [var-annotated]`
- Runtime type errors that could be caught at development time

## Rule
**ALWAYS provide explicit type annotations for:**

1. **All variables** - especially collections (lists, dicts, sets)
2. **Function parameters** - with proper typing imports
3. **Function return types** - even if they return None
4. **Class attributes** - in class definitions

## 🚫 AVOID `Any` - Use Proper Types Instead

**CRITICAL:** Never use `Any` when the actual type is available. This defeats the purpose of type annotations.

### Instead of `Any`, use:
- **Third-party library types**: Import proper types from libraries (e.g., `from google.genai.types import File`)
- **Built-in types**: Use `str`, `int`, `bool`, `list[str]`, `dict[str, int]`, etc.
- **Union types**: Use `str | None` instead of `Any` when multiple types are valid
- **Protocol/ABC**: Create proper interfaces for complex objects

### When `Any` might be acceptable:
- **Legacy integrations** where types are truly unknown
- **Dynamic JSON processing** (but prefer TypedDict when structure is known)
- **Temporary during migration** (with TODO comments to fix)

## Examples

### ❌ Bad (Missing type annotations and using Any)
```python
from typing import Any

# Missing variable annotation
warnings = []
data = {}
results = None

# Using Any instead of proper types
uploaded_file: Any = client.files.upload(file=f)  # BAD!
response: Any = client.generate_content(prompt)    # BAD!

# Missing function annotations  
def process_file(path, options):
    return None

# Missing class attribute annotations
class Config:
    def __init__(self):
        self.settings = {}
```

### ✅ Good (Proper type annotations with specific types)
```python
from typing import Optional
from google.genai.types import File as GenAIFile, GenerateContentResponse

# Proper variable annotations with specific types
warnings: list[str] = []
data: dict[str, str] = {}
results: Optional[ProcessingResult] = None

# Using proper third-party types instead of Any
uploaded_file: GenAIFile = await client.aio.files.upload(file=f)
response: GenerateContentResponse = await client.generate_content(prompt)

# Proper function annotations
def process_file(path: str, options: dict[str, str]) -> Optional[ProcessingResult]:
    return None

# Proper class attribute annotations  
class Config:
    def __init__(self) -> None:
        self.settings: dict[str, str] = {}
```

## Required Imports
Always import necessary typing utilities:
```python
from typing import (
    Any, Dict, List, Optional, Union, 
    Literal, Tuple, Set, Callable
)
from enum import Enum
from pydantic import BaseModel, Field  # For Pydantic models
```

## Modern Type Annotations (Python 3.10+)
**⚠️ IMPORTANT**: This rule covers the requirement for type annotations. For the specific syntax to use, see the companion rule: **`python-modern-type-annotations.mdc`**

For Python 3.10+ projects, use modern syntax:
- `dict[str, int]` instead of `Dict[str, int]`
- `list[str]` instead of `List[str]`
- `str | None` instead of `Optional[str]`

## Checklist for the Assistant
- [ ] All variables have explicit type annotations
- [ ] All function parameters have type annotations  
- [ ] All function return types are annotated (including `-> None`)
- [ ] All class attributes are properly typed
- [ ] **NO use of `Any`** - use proper types from libraries/built-ins instead
- [ ] Import specific types from third-party libraries (e.g., `from google.genai.types import File`)
- [ ] Use **modern Python 3.10+ syntax** (see `python-modern-type-annotations.mdc`)
- [ ] Required typing imports are present (minimize with modern syntax)

This ensures mypy compliance and prevents `[var-annotated]` and similar type checking errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JPH44-pro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
