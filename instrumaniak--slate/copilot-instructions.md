## slate

> The code and tests MUST NEVER crash, freeze, or consume excessive host resources. All work must implement these safeguards:

# Slate - GTK4 Code Editor


## ⚠️ SYSTEM SAFETY MANDATES

The code and tests MUST NEVER crash, freeze, or consume excessive host resources. All work must implement these safeguards:

### Memory Safety
- **ALL tests must have timeouts**: Use `@pytest.mark.timeout(N)` decorator (N ≤ 60s for fast tests, ≤ 120s for slow)
- **Never run full pytest without safeguards** - can freeze system if tests have memory leaks or infinite loops
- **Always use fixtures that cleanup after themselves** - temp directories, orphan processes
- **Test with resource limits**: Use `resource.setrlimit()` to cap memory consumption in test fixtures

### Process Safety
- **Kill orphan processes after each test** - especially GTK apps, subprocess spawns
- **Use fixtures with autouse teardown** to kill leaked processes (e.g., slate, gtk, gdbus)
- **Never leave subprocesses hanging** - always use `finally` or `yield` to terminate
- **E2E/subprocess tests must use `@pytest.mark.slow @pytest.mark.e2e`** - allow exclusion via `-m "not e2e"`

### Test Configuration (Mandatory in pyproject.toml)
```toml
[tool.pytest.ini_options]
timeout = 60
timeout_hard_timeout = 120
```

### Test Safety Rules
1. **Never** run `pytest tests/` without understanding which tests spawn GTK/subprocess
2. **Always** use `--ignore=tests/e2e/` unless you have D-Bus + xvfb + AT-SPI
3. **Safe default**: `pytest tests/core/ tests/services/ -v` (no GTK, no subprocess)
4. **Use markers**: `@pytest.mark.slow`, `@pytest.mark.e2e`, `@pytest.mark.gtk` for filtering

---

## Agent Workflow

### Task Management
- Break complex tasks into smaller steps and track with `todowrite`
- Create todo items with `status: in_progress`, `pending`, or `completed`
- Keep one todo `in_progress` at a time — complete before starting new

### Research & Verification
- Use `websearch` or `codesearch` when uncertain about libraries, APIs, or patterns
- Verify findings by checking existing code in `slate/`, `tests/`, or `pyproject.toml`
- Check system dependencies: `dpkg -l | grep -E "python3-gi|gtk|gtksource"` before assuming packages missing
- Cross-reference with Context7 MCP for framework-specific questions

### Subagent Delegation
- Use `task` tool for parallel work on independent files/components
- Never delegate overlapping file changes to multiple subagents simultaneously
- Before delegating: verify which files the subagent will modify to avoid conflicts
- Combine related file changes in single agent to maintain context

### Quality & Performance
- Run `ruff check` + `ruff format` after every code change
- Run `mypy slate/` before considering work complete
- Test at least one logical path end-to-end (not just unit tests)
- For multi-file changes: verify lint/typecheck pass before moving on

## Commands

```bash
# Development setup
pip install -e ".[dev]"

# Lint & format
ruff check slate/ tests/
ruff format .

# Type check
mypy slate/

# Safe test (core + services only - no GTK/subprocess)
pytest tests/core/ tests/services/ -v

# Full tests (requires xvfb + dbus for E2E)
pytest tests/ -v -m "not e2e"
xvfb-run -a pytest tests/ -v

# Run app
slate                    # open empty editor
slate /path/to/file      # open file on startup
```

## Architecture

```
slate/
├── core/       # Dataclasses, EventBus, plugin API (ZERO GTK imports)
├── services/   # FileService, GitService, ConfigService (lazy GTK imports)
├── ui/         # GTK4 widgets, main window, editor (direct GTK)
└── plugins/    # File explorer, source control (depends only on core)
```

**Layer rules**: `core` → `services` → `ui` → `plugins`. Lower layers never import from higher.

## Critical Anti-Patterns

### Resource Safety (NEVER DO)
- ❌ Never leave orphan processes - always cleanup in fixtures with `finally` or `yield`
- ❌ Never run full pytest without `--ignore=tests/e2e/` unless you have proper display setup
- ❌ Never test GTK without xvfb or display server - will freeze system
- ❌ Never assume tests don't spawn subprocesses - always verify and cleanup

### Code Safety
- ❌ Never `from gi.repository import Gtk` at module level in `core/` or `services/`
- ❌ Never call `gitpython` or `open()` from UI layer — use services
- ❌ Never emit `FileOpenedEvent` directly — only `TabManager` does this
- ❌ Never show windows or mutate tabs in plugin `activate()`
- ❌ Never use GTK signals for cross-component communication — use `EventBus`
- ❌ Never configure `GtkSource.View` directly — use `EditorViewFactory`

### Memory Leak Prevention (T003)
- ❌ **Never set Python instance attributes on GObject wrappers** (`Gtk.*`, `GtkSource.*`, etc.). Setting `self.x = ...` switches PyGObject (≤3.42) into toggle-ref mode, pinning the wrapper + C object alive once the widget is realized → leak. Store state externally (module `WeakKeyDictionary`, `id()`-keyed dict, or locals). Reference fix: `EditorView` in `slate/ui/editor/editor_view.py`.
- ❌ **Never `connect()` a bound method as a signal handler on a signal that outlives the widget** — the closure pins the widget. Connect a closure that derefs `weakref.ref(self)` and no-ops when the view is gone.
- ❌ **Never stash GObjects as widget attributes just to keep them alive** (e.g. `view._slate_font_css_provider`). C APIs like `add_provider()` already hold their own ref; the Python attr only adds a retention path.
- ✅ **Regression pattern**: any widget with signal connections gets a probe test like `tests/ui/gtk/test_tab_memory.py` — count widget instances via a `gc.get_objects()` *generator* (never list comprehension) across N open→close cycles, assert count returns to baseline. Flat instance count + slow RSS growth = allocator retention, **not** a leak.
- ✅ **pygobject constraint**: this system runs apt `python3-gi` 3.42.1 (toggle-ref leak fixed upstream in 3.56). Do NOT upgrade via `pip --user` (shadows system bindings for all apps); if 3.56 is ever wanted, use a dedicated venv.

## Testing

### Test Fixtures (Mandatory)
All test directories must have a `conftest.py` with these safeguards:

```python
import gc
import resource

import pytest

# 1. Memory limit via resource.setrlimit(resource.RLIMIT_AS, (2GB, hard_limit))

# 2. GC after each test
def pytest_runtest_teardown(item, nextitem):
    gc.collect()

# 3. Kill orphan processes
@pytest.fixture(autouse=True)
def cleanup_orphan_processes():
    yield
    # kill leaked slate/gtk processes
```

### Test Markers (Required)
- `@pytest.mark.timeout(N)` - ALL tests must timeout
- `@pytest.mark.slow` - tests that spawn subprocess or GTK
- `@pytest.mark.e2e` - end-to-end tests requiring display/desktop

### Coverage Requirements
- `core/` and `services/`: 90%+ coverage required
- `ui/`: smoke/integration tests only (no widget coverage chase)
- Use `temp_dir`, `temp_git_repo` fixtures over excessive mocking
- GTK tests need `pytest-xvfb` or display server

## System Dependencies

```bash
sudo apt-get install python3-gi python3-gi-cairo gir1.2-gtk-4.0 gir1.2-gtksource-5 git ripgrep
```

## Key Files

- Entry: `slate/main.py` → `slate/ui/app.py`
- Config: `~/.config/slate/config.ini`
- Tests mirror source: `tests/core/`, `tests/services/`, `tests/ui/`, `tests/plugins/`

## Reference

Comprehensive rules: `_bmad-output/project-context.md`

---
> Source: [instrumaniak/slate](https://github.com/instrumaniak/slate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
