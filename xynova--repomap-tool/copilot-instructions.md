## rules-for-python-coding

> Python Coding Rules and Best Practices


# 🐍 **PYTHON CODING STANDARDS**

## 🎯 **CORE PRINCIPLES**
- **DRY (Don't Repeat Yourself)**: Eliminate code duplication through abstraction and reuse
- **Type Safety**: Full mypy compatibility with comprehensive type annotations
- **Data Validation**: Use Pydantic models for all data structures and validation
- **Code Quality**: Format with black, lint with flake8, type-check with mypy
- **Modern Python**: Use Python 3.11+ features and best practices

### **Language Standards:**
- **ALWAYS** write all code (comments, variable names, function names, etc.) exclusively in English.
- **NEVER** use non-English characters or words in any part of the codebase.

## 📝 **CODE FORMATTING REQUIREMENTS**

### **MANDATORY FORMATTING STEPS:**
1. **ALWAYS** run `make format` before claiming work is complete
2. **ALWAYS** run `make lint` to check for style issues
3. **ALWAYS** run `make mypy` to verify type safety
4. **NEVER** submit unformatted code
5. **NEVER** ignore linting warnings or errors

### **Formatting Tools:**
- **Black**: Code formatting (line length: 88 characters)
- **Flake8**: Linting with custom rules for this project
- **MyPy**: Static type checking with strict mode
- **isort**: Import sorting (handled by black)

## 🔄 **DRY (DON'T REPEAT YOURSELF) PRINCIPLES**

### **Code Reuse Requirements:**
- **ALWAYS** extract common functionality into reusable functions/classes
- **ALWAYS** use inheritance and composition to avoid duplication
- **ALWAYS** create utility functions for repeated logic
- **ALWAYS** use decorators for cross-cutting concerns
- **NEVER** copy-paste code blocks
- **NEVER** duplicate validation logic
- **NEVER** repeat configuration patterns

### **DRY Implementation Patterns:**
```python
# ✅ GOOD: Reusable base class
class BaseMatcher(ABC):
    @abstractmethod
    def match(self, query: str, identifiers: List[str]) -> List[MatchResult]:
        pass

# ✅ GOOD: Utility function
def validate_threshold(value: float) -> float:
    if not 0.0 <= value <= 1.0:
        raise ValueError(f"Threshold must be between 0.0 and 1.0, got {value}")
    return value

# ❌ BAD: Repeated validation logic
def fuzzy_search(self, query: str) -> List[MatchResult]:
    if not 0.0 <= self.threshold <= 1.0:  # Duplicated
        raise ValueError("Invalid threshold")
    # ...

def semantic_search(self, query: str) -> List[MatchResult]:
    if not 0.0 <= self.threshold <= 1.0:  # Duplicated
        raise ValueError("Invalid threshold")
    # ...
```

## 🏷️ **TYPE ANNOTATIONS & MYPY COMPATIBILITY**

### **Type Annotation Requirements:**
- **ALWAYS** annotate function parameters and return types
- **ALWAYS** annotate class attributes
- **ALWAYS** use `from __future__ import annotations` for forward references
- **ALWAYS** use `typing` module for complex types
- **ALWAYS** use `typing_extensions` for newer type features
- **NEVER** use `Any` without explicit justification
- **NEVER** ignore mypy errors

### **Type Annotation Examples:**
```python
from __future__ import annotations
from typing import List, Dict, Optional, Union, Protocol, TypeVar
from typing_extensions import Self

# ✅ GOOD: Comprehensive type annotations
class SearchEngine:
    def __init__(self, config: RepoMapConfig) -> None:
        self.config: RepoMapConfig = config
        self.cache: Dict[str, List[MatchResult]] = {}
    
    def search(
        self, 
        query: str, 
        identifiers: List[str],
        max_results: Optional[int] = None
    ) -> List[MatchResult]:
        # Implementation
        pass
    
    def get_cache_stats(self) -> Dict[str, Union[int, float]]:
        return {"size": len(self.cache), "hit_rate": 0.85}

# ✅ GOOD: Generic types
T = TypeVar('T')

class Cache(Generic[T]):
    def get(self, key: str) -> Optional[T]:
        pass
    
    def set(self, key: str, value: T) -> None:
        pass

# ✅ GOOD: Protocol for duck typing
class Matcher(Protocol):
    def match(self, query: str, identifiers: List[str]) -> List[MatchResult]:
        ...
```

### **MyPy Configuration Compliance:**
- **ALWAYS** use strict mode settings
- **ALWAYS** handle `Optional` types explicitly
- **ALWAYS** use `Union` types when multiple types are possible
- **ALWAYS** use `Literal` types for string/enum-like values
- **ALWAYS** use `Final` for constants

## 🏗️ **PYDANTIC MODEL REQUIREMENTS**

### **Data Model Standards:**
- **ALWAYS** use Pydantic models for data validation
- **ALWAYS** define field types and constraints
- **ALWAYS** use validators for complex validation logic
- **ALWAYS** use `model_config` for Pydantic v2 settings
- **ALWAYS** use `Field()` for field metadata
- **NEVER** use plain dataclasses for validated data
- **NEVER** skip validation for user input

### **Pydantic Model Examples:**
```python
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import List, Optional, Literal
from enum import Enum

class MatchType(str, Enum):
    FUZZY = "fuzzy"
    SEMANTIC = "semantic"
    HYBRID = "hybrid"

class MatchResult(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True,
        extra="forbid"
    )
    
    identifier: str = Field(..., min_length=1, description="The matched identifier")
    score: float = Field(..., ge=0.0, le=1.0, description="Match confidence score")
    match_type: MatchType = Field(..., description="Type of matching used")
    line_number: Optional[int] = Field(None, ge=1, description="Line number in source")
    file_path: str = Field(..., description="Path to the source file")
    
    @field_validator('identifier')
    @classmethod
    def validate_identifier(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Identifier cannot be empty or whitespace only")
        return v.strip()

class SearchRequest(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True
    )
    
    query: str = Field(..., min_length=1, max_length=1000)
    match_type: MatchType = Field(default=MatchType.FUZZY)
    threshold: float = Field(default=0.7, ge=0.0, le=1.0)
    max_results: int = Field(default=50, ge=1, le=1000)
    project_path: Optional[str] = Field(None, description="Project root path")
    
    @field_validator('query')
    @classmethod
    def validate_query(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Query cannot be empty")
        return v.strip()
```

## 🏛️ **ARCHITECTURE PATTERNS**

### **Class Design Principles:**
- **ALWAYS** use single responsibility principle
- **ALWAYS** prefer composition over inheritance
- **ALWAYS** use dependency injection for testability
- **ALWAYS** implement proper `__repr__` and `__str__` methods
- **ALWAYS** use context managers for resource management
- **NEVER** create god classes with multiple responsibilities
- **NEVER** use global state

### **Error Handling Standards:**
```python
from typing import NoReturn
import logging

# ✅ GOOD: Custom exception hierarchy
class RepoMapError(Exception):
    """Base exception for RepoMap tool."""
    pass

class ValidationError(RepoMapError):
    """Raised when data validation fails."""
    pass

class ConfigurationError(RepoMapError):
    """Raised when configuration is invalid."""
    pass

# ✅ GOOD: Proper error handling with logging
def process_file(file_path: Path) -> List[MatchResult]:
    try:
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        return extract_identifiers(content)
    except FileNotFoundError:
        logger.error(f"File not found: {file_path}")
        raise ValidationError(f"File not found: {file_path}")
    except UnicodeDecodeError as e:
        logger.error(f"Unicode decode error in {file_path}: {e}")
        raise ValidationError(f"Invalid file encoding: {file_path}")

### **Error Suppression Anti-Patterns:**

- **NEVER** use `try/except/pass` - Completely ignores errors (security risk)
- **NEVER** use `try/except/continue` without logging - Suppresses iteration errors  
- **ALWAYS** log errors at appropriate levels instead of suppressing them
- **ALWAYS** use specific exception types instead of blanket `Exception`
- **NEVER** completely silence errors - Maintain observability for debugging

**Examples:**
```python
# ❌ BAD (security risk):
try:
    operation()
except Exception:
    pass  # Error completely hidden

# ❌ BAD (poor observability):  
try:
    operation() 
except Exception:
    continue  # Error suppressed during iteration

# ✅ GOOD (secure and observable):
try:
    operation()
except (SpecificError, ExpectedError) as e:
    logger.debug(f"Operation failed gracefully: {e}")
    # Handle or continue appropriately
```

### **Resource Management Standards:**

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
