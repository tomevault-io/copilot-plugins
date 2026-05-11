## jsrun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jsrun is a Python library providing JavaScript runtime capabilities via Rust and V8. It exposes a Python API for executing JavaScript code in isolated V8 contexts with async support and permission controls.

**Tech Stack:**
- Rust (core runtime using deno_core)
- Python bindings via PyO3
- Build: Maturin for Python-Rust integration
- Testing: pytest with pytest-asyncio

## Build & Development Commands

The project uses a Makefile for common development tasks. Run `make help` to see all available commands.

### Initial setup
```bash
# Install dependencies and dev tools
make install
```

### Build the project
```bash
# Build development version (debug mode)
make build-dev
```

### Run tests
```bash
# Run all Python tests
make test

# Run tests quietly
make test-quiet

# Run specific test file or pattern
uv run pytest tests/test_runtime.py

# Run tests with asyncio support
uv run pytest tests/test_runtime.py::TestRuntimeAsync -v

# Run Rust tests
cargo test

# Run a single Rust test
cargo test test_runtime_lifecycle
```

### Linting and formatting
```bash
# Auto-format both Rust and Python code
make format

# Lint all code (Rust + Python)
make lint

# Lint Python only
make lint-python

# Auto-fix Python linting issues
make lint-python-fix

# Lint Rust only
make lint-rust
```

### Documentation
```bash
# Build documentation
make docs

# Serve docs locally with live reload
make docs-serve
```

### Standard CI workflow
```bash
# Run the full CI pipeline locally (format, build, lint, test)
make all
```

### Cleanup
```bash
# Remove build artifacts and caches
make clean
```

## Architecture

### Multi-Layer Design

The project has three distinct layers that communicate via well-defined boundaries:

1. **Rust Core** (`src/runtime/`): V8 isolate management, async execution, ops system
2. **Rust-Python Bridge** (`src/runtime/python.rs`, `src/lib.rs`): PyO3 bindings
3. **Python API** (`python/jsrun/__init__.py`): User-facing interface

### Type Conversion Notes

- JavaScript `undefined` now round-trips via the `JsUndefined` sentinel (`jsrun.undefined`), distinct from Python `None` / JS `null`.
- Binary types (`Uint8Array`, `ArrayBuffer`) map to Python `bytes`; Python `bytes`, `bytearray`, and `memoryview` map back to `Uint8Array`.
- Temporal values (`Date` ↔ `datetime`), sets (`Set` ↔ `set`), and arbitrary precision integers (`BigInt` ↔ Python `int`) are handled natively, including op arguments/results.

### Threading Model

Each JavaScript runtime runs on a **dedicated OS thread** with its own:
- V8 isolate (single-threaded, non-Send)
- Tokio single-threaded runtime for async operations
- Command channel for host communication

The main Python thread communicates with runtime threads via message passing (`HostCommand` enum in `runner.rs`).

### Key Components

**RuntimeHandle** (`src/runtime/handle.rs`):
- Clone-safe handle to a runtime thread
- Sends commands via async_mpsc channel
- Does NOT auto-shutdown on drop (explicit `close()` required)
- Thread-safe via Arc\<Mutex\> for shutdown state

**Runtime Thread** (`src/runtime/runner.rs`):
- `RuntimeCoreState` holds the V8 isolate (deno_core JsRuntime) and all runtime data
- `RuntimeDispatcher` processes commands from host thread on the dedicated runtime thread
- Handles promise polling with microtask checkpoints
- Manages JavaScript event loop execution

**Ops System** (`src/runtime/ops.rs`):
- Permission-based host function registry
- Sync and async ops with JSON serialization
- JavaScript calls ops via `__host_op_sync__()` and `__host_op_async__()`

**Module System** (`src/runtime/loader.rs`):
- Static module registration via `add_static_module()`
- Custom module resolution and loading
- Support for both sync and async module evaluation
- ES module imports/exports

**Inspector/Debugger** (`src/runtime/inspector.rs`):
- Chrome DevTools protocol server for debugging
- WebSocket-based inspector sessions
- Runs on dedicated thread with own event loop
- Exposes metadata for connecting devtools frontend

**Snapshot Builder** (`src/runtime/snapshot.rs`):
- Pre-initialize V8 heap state for faster startups
- Execute bootstrap code once at snapshot time
- Create runtime instances from snapshot
- Useful for serverless/multi-tenant scenarios

**Streaming Bridge** (`src/runtime/stream.rs`):
- Bidirectional stream conversion (JS ReadableStream ↔ Python async iterables)
- Non-blocking chunk transfer with backpressure
- Automatic lifecycle management and cleanup
- Stream stats tracking for monitoring

### V8 Platform Initialization

V8 requires exactly one global platform instance. The code uses `OnceCell` to ensure `initialize_platform_once()` is safe to call multiple times. Always call this before creating runtimes (it's done automatically in `Runtime()`).

## Important Implementation Details

### Promise Handling

Async evaluation (`eval_async`) works by:
1. Executing the code and checking if result is a promise
2. If promise: poll via repeated `perform_microtask_checkpoint()` + `yield_now()`
3. Check promise state (Pending/Fulfilled/Rejected)
4. Optional timeout enforced via `tokio::time::timeout`

### Op Registry and Embedder Data

Each V8 context stores a pointer to the shared `OpRegistry` in embedder slot 0. This allows JavaScript callback functions to access the registry without additional state passing. The pointer must be properly cleaned up (converted back to `Rc` and dropped) to prevent leaks.

### Python-Rust-JS Data Flow

1. Python calls `runtime.eval("code")`
2. PyO3 converts to Rust `String`
3. `RuntimeHandle` sends `HostCommand::Eval` via channel
4. Runtime thread receives command, compiles V8 script
5. Result converted to string, sent back via sync channel
6. PyO3 converts to Python `str`

For ops: JS → Rust callback → JSON → Python function → JSON → Rust → JS promise resolution

### Error Handling

- Rust errors: `Result<T, String>` (error messages as strings)
- Python exceptions: Converted to `PyRuntimeError` at boundary
- JavaScript exceptions: Caught and converted to Rust `Err`

### Context-Local Runtime Management

The `python/jsrun/__init__.py` module provides convenience functions (`jsrun.eval()`, `jsrun.bind_function()`) that use a context-local runtime:

1. Each asyncio task or thread gets its own isolated `Runtime` instance
2. Stored in `contextvars.ContextVar` for per-task isolation
3. Automatically created on first use, cleaned up when task/thread completes
4. Accessed via `get_default_runtime()` or implicitly via module-level functions

This enables simple usage without manual runtime lifecycle management while maintaining isolation between concurrent requests/tasks.

### Inspector Architecture

The inspector (`src/runtime/inspector.rs`) runs on a separate thread from runtime threads:

1. Inspector thread runs its own single-threaded Tokio runtime
2. Hosts HTTP/WebSocket server for Chrome DevTools Protocol
3. Each runtime can have one inspector session
4. Inspector forwards CDP messages to V8's inspector via `JsRuntimeInspector`
5. Useful for debugging, profiling, and understanding runtime behavior

### Snapshot Implementation

Snapshots use `deno_core::JsRuntimeForSnapshot` to capture V8 heap state:

1. Create builder with optional bootstrap script
2. Execute initialization code (libraries, polyfills, etc.)
3. Call `create_snapshot()` to serialize heap to bytes
4. Pass snapshot to `RuntimeConfig` when creating new runtimes
5. New runtimes start with pre-initialized state, skipping bootstrap

Snapshots reduce cold-start time for frequently-used libraries or configurations.

## Testing Structure

**Rust tests** (`#[cfg(test)]` blocks in each module):
- Unit tests for core components
- Integration tests for runtime lifecycle
- Tests for ops, contexts, async execution

**Python tests** (`tests/`):
- `test_runtime.py`: Comprehensive integration tests
- Tests organized by feature (basics, async, timeout, concurrency)
- Use pytest fixtures and context managers

When adding features, add tests at both layers.

## Documentation Structure

The project uses MkDocs with Material theme for documentation:

- `docs/index.md`: Landing page and introduction
- `docs/quickstart.md`: Getting started guide
- `docs/concepts/`: Core concepts like runtime model and type conversion
- `docs/use-cases/`: Real-world usage examples (playground, etc.)
- `docs/api/`: Auto-generated API reference from docstrings
- `docs/internals/`: Architecture deep-dives
- `docs/stubs/`: Type stubs for mkdocstrings to generate API docs
- `mkdocs.yml`: Site configuration

When adding new features, update relevant documentation in `docs/`. API documentation is generated from Python docstrings using mkdocstrings.

## Common Patterns

### Using the context-local runtime API
```python
import jsrun

# The easiest way - automatic per-task/thread isolation
result = jsrun.eval("2 + 2")

# Bind Python functions to JavaScript
jsrun.bind_function("notify", lambda msg: print("JS:", msg))
jsrun.eval("notify('hello')")

# Bind Python objects to JavaScript
jsrun.bind_object("config", {"debug": True, "version": "1.0"})
jsrun.eval("config.version")

# Get explicit access to the context-local runtime
runtime = jsrun.get_default_runtime()
```

### Spawning a runtime explicitly (Python)
```python
from jsrun import Runtime

with Runtime() as runtime:
    result = runtime.eval("2 + 2")
```

### Async evaluation (Python)
```python
import asyncio

async def main():
    with Runtime() as runtime:
        result = await runtime.eval_async(
            "Promise.resolve(42)",
            timeout=1.0
        )
```

### Binding Python functions to JavaScript
```python
from jsrun import Runtime

with Runtime() as runtime:
    # Bind a simple function
    def add(a, b):
        return a + b

    runtime.bind_function("add", add)
    result = runtime.eval("add(2, 3)")  # 5

    # Async functions work too
    async def fetch_data(url):
        # ... async operation
        return {"data": "..."}

    runtime.bind_function("fetchData", fetch_data)
    # JS will receive a Promise that resolves when Python completes
```

### Module loading with custom resolver and loader
```python
import asyncio
from jsrun import Runtime

async def main():
    with Runtime() as rt:
        # Static module
        rt.add_static_module("math", "export const answer = 42;")

        # Custom resolver and loader
        def resolver(specifier: str, referrer: str) -> str | None:
            if specifier.startswith("custom:"):
                return specifier
            return None

        async def loader(specifier: str) -> str:
            if specifier == "custom:message":
                return "export const text = 'Hello from custom loader';"
            raise ValueError(f"Unknown module: {specifier}")

        rt.set_module_resolver(resolver)
        rt.set_module_loader(loader)

        # Evaluate module
        namespace = await rt.eval_module_async("entry")
```

### Threading and GIL release
```python
import threading
from jsrun import Runtime

def run_js_in_thread():
    with Runtime() as rt:
        # This releases the GIL, allowing other Python threads to run
        result = rt.eval("Math.sqrt(16)")
        print(f"JS result: {result}")

# Run JS in parallel with Python threads
js_thread = threading.Thread(target=run_js_in_thread)
js_thread.start()
# Other Python threads can make progress while JS runs
js_thread.join()
```

### Inspector for debugging
```python
from jsrun import Runtime, InspectorConfig, RuntimeConfig

# Configure inspector at runtime creation
inspector_config = InspectorConfig(
    display_name="My Runtime",
    wait_for_connection=False
)
config = RuntimeConfig(inspector=inspector_config)

with Runtime(config) as rt:
    # Get inspector endpoints
    endpoints = rt.inspector_endpoints()
    if endpoints:
        print(f"DevTools: {endpoints.devtools_frontend_url}")
        print(f"WebSocket: {endpoints.websocket_url}")

    rt.eval("debugger; console.log('Debug me!')")
```

### Creating snapshots for faster startup
```python
from jsrun import SnapshotBuilder

# Create snapshot with bootstrap code
builder = SnapshotBuilder()
builder.execute_script("myLib.js", "globalThis.myLib = { version: '1.0' };")
snapshot = builder.create_snapshot()

# Use snapshot when creating runtimes
from jsrun import Runtime, RuntimeConfig

config = RuntimeConfig(snapshot=snapshot)
with Runtime(config) as rt:
    # myLib is already available, no need to re-initialize
    result = rt.eval("myLib.version")
```

### Streaming between JavaScript and Python
```python
import asyncio
from jsrun import Runtime

async def main():
    with Runtime() as rt:
        # Python async iterable → JS ReadableStream
        async def data_generator():
            for i in range(5):
                yield {"count": i}
                await asyncio.sleep(0.1)

        stream_id = await rt.create_js_stream_from_python(data_generator())
        rt.eval(f"globalThis.myStream = __jsrun_get_stream__({stream_id})")

        # Consume in JavaScript
        result = await rt.eval_async("""
            const reader = myStream.getReader();
            const chunks = [];
            while (true) {
                const {done, value} = await reader.read();
                if (done) break;
                chunks.push(value);
            }
            chunks
        """)
        print(result)  # List of chunks

asyncio.run(main())
```

## File Organization

- `src/lib.rs`: PyO3 module definition, exception types
- `src/runtime/mod.rs`: V8 platform initialization, re-exports
- `src/runtime/runner.rs`: Thread spawning, event loop, RuntimeCoreState and RuntimeDispatcher
- `src/runtime/handle.rs`: RuntimeHandle API
- `src/runtime/python/`: Python bindings split into modules:
  - `runtime.rs`: Python Runtime class
  - `bridge.rs`: JS-Python async bridges
  - `error.rs`: Error conversion utilities
  - `stats.rs`: Runtime statistics exports
  - `snapshot.rs`: Snapshot builder bindings
- `src/runtime/config.rs`: Configuration builder
- `src/runtime/ops.rs`: Op registry and permissions
- `src/runtime/loader.rs`: Module loading and resolution
- `src/runtime/inspector.rs`: Chrome DevTools protocol server
- `src/runtime/snapshot.rs`: Snapshot creation and management
- `src/runtime/stream.rs`: Streaming bridge (JS ↔ Python)
- `python/jsrun/__init__.py`: Python package with context-local runtime API
- `tests/`: Python integration tests
- `examples/`: Usage examples including threading, modules, inspector
- `docs/`: MkDocs documentation site

## Common Pitfalls

1. **Not closing runtimes**: When using `Runtime()` explicitly, always use context manager or call `.close()`. The context-local API (`jsrun.eval()`) handles cleanup automatically.

2. **Op permission mismatches**: Ops requiring permissions will fail if runtime not granted those permissions via `RuntimeConfig`.

3. **Infinite promises**: Using `eval_async` without timeout on never-resolving promises will hang. Always consider timeout for untrusted code.

4. **Module resolution order**: Custom resolver is checked first, then static modules. Return `None` from resolver to fall back to static modules.

5. **Streaming lifecycle**: JavaScript streams created from Python iterables must be consumed completely or explicitly released to avoid resource leaks.

6. **Context-local runtime confusion**: Each asyncio task and thread gets its own isolated runtime via `jsrun.eval()`. If you need shared state, use `Runtime()` explicitly and pass it around.

7. **Inspector blocking**: Setting `wait_for_connection=True` in InspectorConfig will pause execution until DevTools connects. Use `False` for non-blocking debugging.

---
> Source: [imfing/jsrun](https://github.com/imfing/jsrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
