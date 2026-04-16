## paravalid

> - **Project:** paravalid

# CLAUDE.md — paravalid

## Purpose
- **Project:** paravalid  
- **Tagline:** Parallel validation and serialization for Python 3.13+  
- **Goal:** In Python 3.13+ free-threaded (no-GIL) builds, provide:
  1) Parallel JSON list serialization/deserialization  
  2) Parallel Pydantic(v2) model validation  
  with automatic sequential fallback when no-GIL is unavailable.

## POC Results (validated)
- **Env:** Python 3.14.0rc2 (free-threaded), GIL disabled, 8 workers  
- **JSON serialization:** **4.66×** faster  
- **Pydantic validation (5,000 users):** **3.36×** faster

## Public API (main)
- `parallel_json_dumps(obj_list, workers='auto', min_size=100, **json_kwargs) -> list[str|Exception]`
- `parallel_json_loads(json_list, workers='auto', min_size=100, **json_kwargs) -> list[Any|Exception]`
- `parallel_validate(model_or_type, data_list, workers='auto', min_size=100, chunk_size=None, **opts) -> list[T|Exception]`
- `is_nogil_available() -> bool`
- `is_nogil_active() -> bool`

### Operating principles
- Parallelize only when **no-GIL is active**; otherwise **sequential fallback**.  
- Use sequential path for small workloads (`len(data) < min_size`).  
- Use **ThreadPoolExecutor** (never ProcessPool).  
- Pydantic validates by **chunks**, results returned in **original order**.  
- Error policy: default `errors='raise'`; optional `errors='collect'` to interleave exceptions in the result list.  
- Support `timeout` (global upper bound); cancel safely on timeout.

## Directory layout
paravalid/
  paravalid/
    __init__.py
    core.py
    json_parallel.py
    pydantic_parallel.py
    version.py
  tests/
    test_core.py
    test_json.py
    test_pydantic.py
  benchmarks/
    bench_json.py
    bench_pydantic.py
  examples/
    example_json.py
    example_pydantic.py
  README.md
  pyproject.toml
  LICENSE
  .gitignore

## Implementation requirements

### core.py
- `is_nogil_available`: prefer `sysconfig.get_config_var("Py_GIL_DISABLED")==1`; fallback to `sys._is_gil_enabled()` when present. Allow `PARAVALID_ASSUME_NOGIL=1` for tests.  
- `is_nogil_active`: runtime GIL state.  
- `auto_workers('auto'|int)`: use `os.cpu_count()`.  
- `should_parallelize(n_items, min_size, nogil)`: only if no-GIL and `n_items>=min_size`.

### json_parallel.py
- `parallel_json_dumps/loads`:  
  - Materialize input as list and **preserve indices** (stable ordering).  
  - `ThreadPoolExecutor(max_workers=auto_workers(workers))` + `as_completed`.  
  - Support `timeout`, `errors=('raise'|'collect')`.  
  - Sequential fast path uses a simple for-loop to minimize overhead.  
  - Pass through `json_kwargs` (`indent`, `ensure_ascii`, `default`, etc.). Document that thread-safety of custom `default` is user’s responsibility.

### pydantic_parallel.py (v2)
- `parallel_validate`:  
  - If `model` is a `BaseModel` subclass, call `model.model_validate`. Otherwise use `TypeAdapter[T].validate_python`.  
  - `chunk_size='auto'|int|None`: `'auto'` = conservative default, e.g., `len(data)//(workers*6)` with min 1.  
  - Submit chunks; merge results back to original indices.  
  - `errors` and `timeout` match JSON behavior.

## Tests (part of DoD)
- **Functionality:** sequential vs parallel produce equivalent outputs; order preserved; `errors='collect'` works.  
- **Environments:** pass with and without no-GIL (performance not guaranteed in the latter; functional parity required).  
- **Pydantic:** cover BaseModel/TypeAdapter paths, nested models, mixed successes/failures.  
- **Edges:** small `N` uses sequential; safe cancellation on timeout.

### Minimum test files
- `tests/test_core.py`: nogil detection, auto_workers, should_parallelize.  
- `tests/test_json.py`: dumps/loads ordering, exceptions, kwargs, timeout.  
- `tests/test_pydantic.py`: model/type-adapter paths, chunking, exceptions, timeout.

## Benchmarks
- `bench_json.py` / `bench_pydantic.py` use `pyperf`; sweep `N` and `workers`.  
- Update README with tables/plots reproducing POC trends.

## Documentation (README)
- Install: `pip install paravalid` (provide `[pydantic]` extra).  
- Quick start examples for JSON and Pydantic.  
- Explain automatic fallback when no-GIL is absent and the small-data sequential path.  
- Document `errors`, `timeout`, `chunk_size`.  
- **Caveats:** user-supplied `json.default` and any global state touched by validators must be thread-safe.

## Quality & style
- Type hints, `from __future__ import annotations`.  
- Minimize external runtime deps (pydantic is optional).  
- Logging off by default; allow lightweight logs with `PARAVALID_DEBUG=1` if needed.

## Prohibited / cautions
- **Do not use ProcessPoolExecutor.**  
- Do not share global mutable state across threads.  
- No additional blocking I/O; this library targets CPU-bound speedups.  
- Avoid tight coupling to unofficial CPython internals; provide fallbacks where necessary.

## Execution order (MVP)
1. Implement `core.py` → unit tests.  
2. Implement `json_parallel.py` → tests.  
3. Implement `pydantic_parallel.py` → tests.  
4. Wire up `__init__.py`, `pyproject.toml` (extras), and minimal README examples.  
5. Add benchmark skeletons → capture results → update README.

## Definition of Done (DoD)
- All tests above pass.  
- Works in both standard CPython and free-threaded builds (install/import/run examples).  
- README “copy-paste” examples run as shown.  
- POC metrics reproduced within reasonable variance.

## Initial commands (terminal)
# Scaffolding / env
uv python install 3.13.7t
uv python pin 3.13.7t
uv add --dev pytest pyperf ruff build

# Run tests (even if empty at first)
uv run pytest -q

# Build (local)
uv build
# uv publish --index pypi   # keep commented until ready

## Claude Code starter prompt (example)
Read the repository’s CLAUDE.md and implement the paravalid MVP according to the DoD.
Start with core.py and tests/test_core.py, make them pass, then implement json_parallel.py and its tests.
After each step passes, proceed to the next one and report changes and commands you ran.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yoon-yoo-sang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
