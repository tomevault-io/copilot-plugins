## sdkgen

> - Use hexagonal/DDD architecture

# SDKGen Coding Guidelines

## Architecture

- Use hexagonal/DDD architecture
- All infrastructure adapters must be dataclasses
- No dependency injection singletons
- Streamlined functional code over imperative
- Domain layer (core/ir.py) has no external dependencies

## Python Style

- No `__future__` imports
- No `abc` package
- Absolute imports only
- Use pathlib (Path) for all file operations (never os.path)
- No underscore prefixes for variables
- TypedDict for data models
- Dataclasses for services/adapters
- Async-first with httpx

## Code Quality

- Target Python 3.12+
- Type hints everywhere (no missing annotations)
- snake_case for Python, camelCase for API
- No useless wrappers
- No variable reassignment (pure functional)
- Consistent return types across similar functions

## Guardian Pattern (Early Returns)

Always use guardian pattern with early returns to reduce nesting:

### ✅ DO: Use early returns for edge cases
```python
def process(data: dict[str, Any]) -> str | None:
    # Guard clauses first
    if not data:
        return None
    if not data.get("valid"):
        return None

    # Main logic at lowest indentation
    return perform_processing(data)
```

### ❌ DON'T: Nest main logic in conditionals
```python
def process(data: dict[str, Any]) -> str | None:
    if data:
        if data.get("valid"):
            # Main logic buried 2+ levels deep
            return perform_processing(data)
    return None
```

**Key principles:**
- Check invalid conditions first and return early
- Keep main logic at the lowest indentation level
- Avoid nesting beyond 2 levels
- Each guard clause should have a clear purpose

## Method Naming (Generated SDKs)

Use 3-priority system for naming generated methods:

### Priority 1: Clean operationId
If operationId present in OpenAPI spec, extract and clean it:
- `create_user_api_v1_users_post` → `create`
- Strip `_api_`, version suffixes (`v1`, `v2`, `beta`)

### Priority 2: RPC-Style Actions
For paths like `/resource/{id}/action`, use action name from this list:

**File operations:** download, upload, export, import
**State changes:** activate, deactivate, enable, disable, publish, unpublish, archive, unarchive
**Workflow:** approve, reject, cancel, complete, submit, confirm, verify, validate
**Execution:** execute, trigger, run, start, stop, pause, resume, retry, restart
**Data operations:** refresh, sync, clone, duplicate, copy, resend, reprocess
**Utility:** summary, status, health, me, current

### Priority 3: HTTP Method + Response Schema
- `GET` + array response → `list()`
- `GET` + object + path param → `get()`
- `GET` + object + no param → use path name (e.g., `health()`, `status()`)
- `POST` → `create()`
- `PUT`/`PATCH` → `update()`
- `DELETE` → `delete()`

## Dataclass Patterns

### ✅ DO: Use field defaults but beware of evaluation time
```python
from dataclasses import dataclass, field

@dataclass
class IRBuilder:
    endpoint_analyzer: EndpointAnalyzer = field(default_factory=EndpointAnalyzer)
    namespace_analyzer: NamespaceAnalyzer = field(default_factory=NamespaceAnalyzer)
```

### ❌ DON'T: Use useless __init__ methods
```python
@dataclass
class IRBuilder:
    def __init__(self):
        self.endpoint_analyzer = EndpointAnalyzer()  # WRONG!
```

## Functional Code Generation

### ✅ DO: Pure functional
```python
return "\n".join([
    "line 1",
    "line 2",
    "line 3",
])
```

### ❌ DON'T: Imperative with mutation
```python
lines = []
lines.append("line 1")
lines.append("line 2")
return "\n".join(lines)
```

## Testing Requirements

- Unit tests for each analyzer and generator
- Integration tests for full pipeline
- Real-world test with test_api
- Verify generated code syntax with `python -m py_compile`
- Type check generated code with `mypy`

## Documentation Standards

- All public methods need docstrings
- Complex logic needs inline comments
- Update docs/ when adding features
- Include examples in docstrings

## Common Pitfalls to Avoid

1. **No inconsistent return types**
   - All generators should return `str`, not mix of `str` and `list[str]`

2. **No variable shadowing**
   - Don't reuse variable names in nested scopes

3. **No truthiness on explicit checks**
   - Use `if value:` not `if value is not None:`

4. **No missing type annotations**
   - Every function parameter and return needs a type hint

5. **No useless wrappers**
   - If a function just calls another function, remove it

6. **No commented-out code**
   - Delete dead code, don't comment it out

7. **Never game the linter**
   - NO `# noqa` comments to suppress warnings
   - NO `# type: ignore` to bypass type checks
   - FIX the actual issue, don't hide it
   - If a rule is genuinely problematic, blacklist it globally in pyproject.toml
   - Never suppress warnings inline

## Generated Code Standards

- All generated files start with `from __future__ import annotations`
- Use `TYPE_CHECKING` for circular import prevention
- Dataclasses for Client and Resources
- TypedDict for models
- Async methods everywhere
- Proper parameter ordering: required first, then optional

## File Organization

```
sdkgen/
├── analyzers/     # Pattern detection (dataclasses)
├── core/          # Application layer + domain models
├── generators/    # Output adapters (dataclasses)
├── utils/         # Shared utilities
└── cli.py         # CLI interface

tests/             # Test suite
test_api/          # FastAPI test server
docs/              # Documentation
```

## Plan-Agent Cycle

After every plan implementation cycle:

1. **Test All Changes**
   - Run all checks: `make check`
   - Or individually:
     - Linter: `make lint`
     - Type checker: `make typecheck`
     - SDK generation: `make test-sdk`
     - Tests: `make test`

2. **Commit Changes with Conventional Commits**
   - Stage all changes: `git add .`
   - Use conventional commit format: `<type>: <description>`
   - Types: feat, fix, docs, style, refactor, perf, test, chore, ci
   - Examples:
     - `feat: add new analyzer for nested resources`
     - `fix: resolve namespace URL doubling bug`
     - `docs: update README with usage examples`
   - Commit: `git commit -m "feat: description of changes"`
   - Push: `git push origin master`
   - Never skip testing before committing

3. **Release-Please Automation**
   - Release-please monitors master branch for conventional commits
   - Automatically creates release PR with version bump and changelog
   - Version bumps: feat=minor, fix=patch, feat!=major
   - Merge release PR to publish to PyPI automatically
   - Check release PRs: `gh pr list`

## When in Doubt

1. Check existing code for patterns
2. Read docs/AGENT.md for comprehensive guidelines
3. Look at test cases for examples
4. Keep it simple and functional

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4thel00z) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
