## feed-spine

> This is FeedSpine, a storage-agnostic, executor-agnostic feed capture framework.

# FeedSpine AI Assistant Rules

# Project Context
This is FeedSpine, a storage-agnostic, executor-agnostic feed capture framework.
Built with Python 3.11+, Pydantic 2.x, async-first, Protocol-based design.

# Critical Rules

## Always Do
- Use `from __future__ import annotations` at top of every file
- Add full type annotations to ALL functions/methods
- Use async for all protocol methods and implementations
- Use **Google-style docstrings** with Args, Returns, Raises, Example
- Include runnable Example (doctest) in ALL public function docstrings
- Mirror test structure to source: `src/feedspine/x/y.py` → `tests/unit/x/test_y.py`
- Use Pydantic v2 syntax (model_config, Field, etc.)
- Raise specific exceptions from feedspine.core.exceptions
- Keep imports organized: stdlib → third-party → local

## Never Do
- Don't use `Optional[T]`, use `T | None` instead
- Don't use `List[T]`, use `list[T]` instead
- Don't mix sync and async in protocols
- Don't use bare `except:` clauses
- Don't skip type annotations
- Don't use relative imports deeper than 2 levels

## Naming Conventions
- Protocols: `StorageBackend`, `CacheBackend`, `SearchBackend`
- Implementations: `MemoryStorage`, `RedisCache`, `PostgresStorage`
- Models: `Record`, `RecordCandidate`, `Sighting`
- Exceptions: `StorageError`, `ValidationError`, `ConfigError`
- Test classes: `TestMemoryStorageBasic`, `TestRecordValidation`
- Test methods: `test_store_new_record_succeeds`, `test_get_missing_returns_none`

## File Patterns

### Protocol file template
```python
"""<Name> protocol."""
from __future__ import annotations
from typing import TYPE_CHECKING, Protocol

class <Name>Backend(Protocol):
    async def initialize(self) -> None: ...
    async def close(self) -> None: ...
```

### Implementation file template
```python
"""<Backend> <concept> implementation."""
from __future__ import annotations
from feedspine.protocols.<concept> import <Concept>Backend

class <Backend><Concept>(<Concept>Backend):
    def __init__(self) -> None:
        self._initialized = False
    
    async def initialize(self) -> None:
        if self._initialized:
            return
        self._initialized = True
    
    async def close(self) -> None:
        self._initialized = False
```

### Test file template
```python
"""Tests for feedspine.<module>.<submodule>."""
from __future__ import annotations
import pytest

@pytest.fixture
async def instance():
    obj = ClassUnderTest()
    await obj.initialize()
    yield obj
    await obj.close()

class TestClassUnderTestBasic:
    async def test_operation_succeeds(self, instance) -> None:
        ...
```

## Directory Structure
```
src/feedspine/
├── core/           # Config, exceptions, logging
├── models/         # Pydantic models
├── protocols/      # Protocol definitions (interfaces)
├── storage/        # StorageBackend implementations
├── search/         # SearchBackend implementations
├── cache/          # CacheBackend implementations
├── blob/           # BlobStorage implementations
├── queue/          # MessageQueue implementations
├── notification/   # Notifier implementations
├── executors/      # Executor implementations
├── pipeline/       # Streaming pipeline
├── workflow/       # DAG workflow
└── reader/         # CLI, API, query service

tests/
├── unit/           # Mirrors src/feedspine/ structure
├── integration/    # Cross-component tests
└── e2e/            # Full system tests
```

## Key Files to Reference
- Models: src/feedspine/models/record.py
- Protocols: src/feedspine/protocols/storage.py  
- Implementations: src/feedspine/storage/memory.py
- Tests: tests/unit/storage/test_memory.py

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryansmccoy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
