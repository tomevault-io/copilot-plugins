## aegis-stack

> **IMPORTANT: When adding new stack configurations (components or services), you MUST add corresponding cache entries to avoid 10+ minute test times!**

# Testing Guide for Aegis Stack

## Stack Cache Fixture (CRITICAL for Test Performance)

**IMPORTANT: When adding new stack configurations (components or services), you MUST add corresponding cache entries to avoid 10+ minute test times!**

### How It Works

The test infrastructure uses a two-tier caching system in `tests/cli/conftest.py`:

1. **`project_template_cache`** (session-scoped): Generates project skeletons ONCE per test session
2. **`project_factory`** (per-test): Copies cached skeletons into fresh temp directories

This avoids regenerating the same project configuration for every test that needs it.

### Cache Configuration: `NAMED_PROJECT_SPECS`

All cached stack configurations are defined in `tests/cli/conftest.py`:

```python
NAMED_PROJECT_SPECS: dict[str, ProjectTemplateSpec] = {
    # Component-based
    "base": ProjectTemplateSpec(),
    "base_with_database": ProjectTemplateSpec(components=("database",)),
    "base_with_scheduler": ProjectTemplateSpec(components=("scheduler",)),
    "base_with_scheduler_sqlite": ProjectTemplateSpec(
        components=("database", "scheduler"), scheduler_backend="sqlite"
    ),
    "base_with_worker": ProjectTemplateSpec(components=("worker",)),
    "base_with_redis": ProjectTemplateSpec(components=("redis",)),
    "scheduler_and_database": ProjectTemplateSpec(components=("database", "scheduler")),
    # Service-based
    "base_with_auth_service": ProjectTemplateSpec(services=("auth",)),
    "base_with_ai_service": ProjectTemplateSpec(services=("ai",)),
    "base_with_ai_sqlite_service": ProjectTemplateSpec(services=("ai[sqlite]",)),
    "base_with_auth_and_ai_services": ProjectTemplateSpec(services=("auth", "ai")),
}
```

### When to Add Cache Entries

**Add a new entry when:**
- You create a new service (e.g., `ai`, `comms`)
- You create a new component combination that tests need
- Multiple tests need the same stack configuration

**Signs you need a new cache entry:**
- Tests are slow because they call `run_aegis_command("init", ...)` directly
- The same stack configuration is generated in multiple tests

### Using the Cache in Tests

```python
from tests.cli.conftest import ProjectFactory

class TestMyFeature:
    def test_something(self, project_factory: ProjectFactory) -> None:
        # Get a cached copy of a base project (fast - just copies files)
        project_path = project_factory("base")

        # Now do your test (e.g., add-service, modify files, etc.)
        result = run_aegis_command("add-service", "auth", "--project-path", str(project_path))

    def test_with_ai(self, project_factory: ProjectFactory) -> None:
        # Get a cached copy of AI service project
        project_path = project_factory("base_with_ai_service")
```

### Performance Impact

- **Without cache:** Each test regenerates project from scratch (~30-40 seconds)
- **With cache:** First test generates, subsequent tests copy (~1-2 seconds)
- **Total impact:** Can reduce test suite from 10+ minutes to 2-3 minutes

### Adding a New Cache Entry

1. **Add to `NAMED_PROJECT_SPECS`** in `tests/cli/conftest.py`:
   ```python
   "base_with_new_service": ProjectTemplateSpec(services=("new_service",)),
   ```

2. **Update tests** to use `project_factory("base_with_new_service")` instead of generating from scratch

3. **Run tests** to verify the cache entry works

---
> Source: [lbedner/aegis-stack](https://github.com/lbedner/aegis-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
