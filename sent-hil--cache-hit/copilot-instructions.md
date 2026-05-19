## cache-hit

> This document contains project-specific guidelines and best practices for Claude Code when working on this codebase.

# CacheHit - Claude Code Guidelines

This document contains project-specific guidelines and best practices for Claude Code when working on this codebase.

## Rules Summary

### 1. Meta - Maintaining This Document

See all rules in section **## META - MAINTAINING THIS DOCUMENT**

1. When adding new rules to detailed sections below, always update this summary section with a corresponding one-sentence summary
2. Each rule in this summary must reference its corresponding detailed section
3. Follow the writing guidelines when adding new rules

### 2. Code Organization

See all rules in section **## CODE ORGANIZATION**

1. Always place imports at the top of the file, never inside functions
2. [Add your rules here]

### 3. Testing

See all rules in section **## TESTING**

1. Always use `patch.object` rather than `patch` for mocking
2. Never use magic numbers or string literals in tests - extract constants
3. Always check for existing test helpers before creating new ones
4. [Add your rules here]

---

## META - MAINTAINING THIS DOCUMENT

### Keeping the Summary Section Up to Date

**Rule**: Whenever you add, modify, or remove rules in the detailed sections below, you MUST update the "Rules Summary" section at the top of this document.

**Process**:

1. Add the new rule to the appropriate detailed section below
2. Add a corresponding one-sentence summary to the Rules Summary section
3. Ensure the summary references the detailed section using the format: "See all rules in section **## SECTION NAME**"
4. If creating a new topic, add both a new numbered topic in the summary AND a new detailed section below

**Example**:
If you add a new rule about async patterns in the detailed "ASYNC PATTERNS" section, you must add:

- A new topic in Rules Summary: "4. Async Patterns - See all rules in section **## ASYNC PATTERNS**"
- A one-sentence summary under that topic

### Writing Effective Guidelines

When adding new rules to this document, follow these principles:

**Core Principles (Always Apply):**

1. **Use absolute directives** - Start with "NEVER" or "ALWAYS" for non-negotiable rules
2. **Lead with why** - Explain the problem/rationale before showing the solution (1-3 bullets max)
3. **Be concrete** - Include actual commands/code for project-specific patterns
4. **Minimize examples** - One clear point per code block
5. **Bullets over paragraphs** - Keep explanations concise
6. **Action before theory** - Put immediate takeaways first

**Optional Enhancements (Use Strategically):**

- **❌/✅ examples**: Only when the antipattern is subtle or common
- **"Why" or "Rationale" section**: Keep to 1-3 bullets explaining the underlying reason
- **"Warning Signs" section**: Only for gradual/easy-to-miss violations
- **"General Principle"**: Only when the abstraction is non-obvious
- **Decision trees**: Only for 3+ factor decisions with multiple considerations

**Anti-Bloat Rules:**

- ❌ Don't add "Warning Signs" to obvious rules (e.g., "imports at top")
- ❌ Don't show bad examples for trivial mistakes
- ❌ Don't create decision trees for simple binary choices
- ❌ Don't add "General Principle" when the section title already generalizes
- ❌ Don't write paragraphs explaining what bullets can convey
- ❌ Don't write long "Why" explanations - 1-3 bullets maximum

---

## TYPE CHECKING WITH ty

We use ty for gradual type checking adoption.

### General Principles

1. **Minimize Logic Changes**: Type checking should NOT change runtime behavior. When adding type hints:

   - Keep original logic intact
   - Use `# type: ignore` comments when necessary
   - Only fix actual type errors, not "improvements"

2. **Test After Type Changes**: Always run tests after adding type hints:
   ```bash
   uv run ty check  # Type checking
   uv run pytest tests/  # Full test suite
   ```

## CODE ORGANIZATION

### Import Placement

**Always place imports at the top of the file, never inside functions.**

```python
# ✅ CORRECT
from datetime import datetime
import pytest

def test_something():
    # test code here
    pass

# ❌ INCORRECT
def test_something():
    from datetime import datetime  # Never inside functions
    pass
```

**Rationale**: Top-level imports make dependencies clear, allow static analysis tools to work correctly, and avoid repeated import overhead.

### Understand lru_cache as Singleton Pattern

**`@lru_cache` on zero-argument functions creates singletons that are populated at startup.**

**Pattern**:

```python
# In another file (e.g., deck_router.py)
@lru_cache
def get_deck_cache() -> Dict[str, Deck]:
    return {}  # Returns same dict instance every call

# In main.py startup
deck_cache = deck_router.get_deck_cache()  # Get the singleton
deck_cache[deck.id] = deck  # Populate it

# In endpoint (first file)
@router.get("/decks")
async def list_decks(cache: Dict[str, Deck] = Depends(get_deck_cache)):
    # Gets the SAME dict that was populated at startup
    return {"decks": list(cache.values())}
```

**What this is NOT**:

- ❌ NOT a cache in the "memoization" sense
- ❌ NOT creating a new instance per request
- ✅ IS a way to create a shared singleton that lives for the app lifetime

**Rationale**: This pattern allows routers to access shared state without globals, while keeping the state
mutable and populated at startup.

---

## TESTING

### Always Use patch.object

**Always use `patch.object` instead of `patch` for mocking.**

```python
# ✅ CORRECT
from unittest.mock import patch

@patch.object(MyClass, 'method_name')
def test_with_mock(mock_method):
    # test code
    pass

# ❌ INCORRECT
@patch('module.path.MyClass.method_name')  # Never use patch with string paths
def test_with_mock(mock_method):
    pass
```

**Rationale**: `patch.object` is more explicit, refactor-safe, and catches errors at test definition time rather than runtime.

### Never Use Magic Numbers or String Literals

**FORBIDDEN: Using literal values scattered throughout tests. Always extract constants and derive values from authoritative sources.**

```python
# ❌ INCORRECT - Magic numbers
def test_timeout():
    assert response.elapsed < 60  # What is 60?
    context = make_context(interval=30)  # What is 30?

# ✅ CORRECT - Named constants
DEFAULT_TIMEOUT_SECONDS = 60
DEFAULT_INTERVAL_SECONDS = 30

def test_timeout():
    assert response.elapsed < DEFAULT_TIMEOUT_SECONDS
    context = make_context(interval=DEFAULT_INTERVAL_SECONDS)
```

**Why It's Wrong**:

- Tests become brittle when constants change
- Unclear where values come from
- Obscures relationships between values

**Rules**:

1. Import constants from source code when possible
2. Define test-specific constants at module level
3. Calculate derived values explicitly to show relationships

### Check for Existing Test Helpers

**Before creating a new test helper or fixture, ALWAYS search for existing ones.**

```bash
# Search for similar helpers:
grep -r "def make_test" tests/ --include="*.py"
grep -r "def create_test" tests/ --include="*.py"
```

**Decision Tree**:

1. Helper exists and fits your needs? → Use it
2. Helper exists but needs extension? → Extend it with optional parameters
3. Helper doesn't exist? → Create it in the appropriate location

**Rationale**: Prevents code duplication, maintains consistency, and makes tests easier to maintain.

### Never Mock Widely-Used Infrastructure

**Never mock widely-used infrastructure (logger, datetime, random, etc.). Instead, create dedicated helper functions that contain the logic, then patch those helpers.**

```python
# ❌ INCORRECT - Patching infrastructure
@patch("loguru.logger.warning")
def test_staleness_logging(mock_warning):
    convert_to_video_moderation_state(result)
    assert mock_warning.called

# ✅ CORRECT - Patch the business logic helper
# In production code:
def log_participant_staleness_warning(user_id: str, channel_id: str, age_seconds: float, ...) -> None:
    """Helper that encapsulates logging logic."""
    logger.warning(
        f"Participant result became stale: {user_id}",
        extra={
            "event": "participant_result_stale",
            "channel_id": channel_id,
            "user_id": user_id,
            "age_seconds": age_seconds,
            ...
        }
    )

# In test code:
@patch.object(conversion_module, "log_participant_staleness_warning")
def test_staleness_logging(mock_log_staleness):
    convert_to_video_moderation_state(result)
    assert mock_log_staleness.called
    assert mock_log_staleness.call_args.kwargs["user_id"] == "test_user_456"
```

**Why It's Wrong to Mock Infrastructure**:

- **Tests implementation, not behavior**: Testing "did we call logger.warning" instead of actual functionality
- **Brittle and coupled**: Changes to logging don't represent functionality changes
- **Infrastructure is everywhere**: Logger used across all files - patching it is fragile
- **Obscures intent**: What behavior are you actually testing?

**General Principle**:

If you find yourself patching widely-used capabilities (logger, datetime, random, database clients), ask: **"Is there a helper/wrapper I should create instead?"**

**Benefits of Helper Functions**:

- Tests verify business logic, not infrastructure calls
- Helper is reusable across the codebase
- Refactoring implementation doesn't break tests
- Type-safe function signature documents what's being done
- Can add additional logic in one place

### Never Mock Model Classes

**Never mock Pydantic model classes or data models.**

```python
# ✅ CORRECT - Create real instances
from gv.ai.common.models.video_moderation_models import VideoModerationResult

def test_with_real_model():
    result = VideoModerationResult(
        channel_id="test_channel",
        user_id="test_user",
        timestamp=datetime.now(UTC),
        details=VideoModerationDetails(
            is_appropriate=True,
            classification="SAFE"
        )
    )
    # test code using real model instance

# ❌ INCORRECT - Don't mock models
from unittest.mock import MagicMock

def test_with_mocked_model():
    result = MagicMock(spec=VideoModerationResult)  # Never do this
    result.channel_id = "test_channel"
```

**Rationale**: Real model instances ensure validation logic is tested and catch schema issues early.

### Test Organization

```
tests/
├── test_persistence_service/
│   ├── test_apis/
│   │   └── test_video_moderation_api.py  # API endpoint tests
│   └── test_models/
│       └── test_video_moderation_models.py  # Model validation tests
├── test_video_moderation_service/
│   ├── test_sampling_integration.py  # Integration tests
│   └── test_video_moderator_sampling.py  # Unit tests for VideoModerator
└── conftest.py  # Shared fixtures
```

### pytest Best Practices

#### Async Tests

```python
# ✅ CORRECT
@pytest.mark.asyncio
async def test_async_function():
    result = await my_async_function()
    assert result is not None
```

#### Parametrized Tests

```python
# ✅ CORRECT - Test multiple cases efficiently
@pytest.mark.parametrize("input_val,expected", [
    (10, 20),
    (5, 10),
    (0, 0),
])
def test_doubling(input_val, expected):
    assert double(input_val) == expected
```

#### Test Naming

```python
# ✅ CORRECT - Descriptive test names
def test_latest_per_user_returns_newest_result_per_user():
    pass

def test_policy_reload_after_60_seconds():
    pass

# ❌ INCORRECT - Vague test names
def test_endpoint():
    pass

def test_policy():
    pass
```

## TESTING - SUMMARY CHECKLIST

Before committing tests, verify:

- [ ] All imports are at the top of the file
- [ ] Using `patch.object` instead of `patch`
- [ ] No mocking of common infrastructure (using testable wrappers instead)
- [ ] No mocking of model classes
- [ ] All test data created using model classes (not dicts/JSON)
- [ ] Checked tests/conftest.py for existing fixtures before creating new ones
- [ ] No naked literals - all timing values, thresholds use constants
- [ ] All tests pass
- [ ] Descriptive test names
- [ ] Edge cases covered
- [ ] No long runs of `=` or other characters for section dividers

---
> Source: [sent-hil/cache-hit](https://github.com/sent-hil/cache-hit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
