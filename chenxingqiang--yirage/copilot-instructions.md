## yirage

> YiRage (Yield Revolutionary AGile Engine) is a multi-backend LLM inference optimization framework — a Python package with C++/Rust/Cython native extensions. It is a standalone compute library (no web servers, databases, or microservices).

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

YiRage (Yield Revolutionary AGile Engine) is a multi-backend LLM inference optimization framework — a Python package with C++/Rust/Cython native extensions. It is a standalone compute library (no web servers, databases, or microservices).

### Building

The C++ build requires `python3-dev` (for `Python.h`). Install via `sudo apt-get install -y python3-dev` if not already present. Build with:

```
YIRAGE_BACKEND=cpu pip install -e . --no-build-isolation
```

Apple Silicon (MPS): `YIRAGE_BACKEND=mps pip install -e . --no-build-isolation`, or `make install-mps` (Makefile exports `PIP_DISABLE_PIP_VERSION_CHECK=1` to avoid post-install SSL noise to PyPI when using proxies).

If pip still prints `SSLEOFError` after a successful install, run `export PIP_DISABLE_PIP_VERSION_CHECK=1` in your shell (or use `make install-cpu` / `make install-mps`).

This compiles Rust libraries, the C++ runtime (`libyirage_runtime.a`), and Cython bindings. The Python package **always** depends on this native runtime: `import yirage` fails if `yirage.core` cannot load (there is no “Python-only” mode). After a successful build, `CORE_AVAILABLE` is true and backends/kernel APIs are available.

### Running the project

- **Lint**: `make lint` or run individual tools: `black --check python/`, `ruff check python/`, `flake8 python/`, `mypy python/yirage --ignore-missing-imports`. See `Makefile` and `pyproject.toml` for full configs.
- **Test (recommended layers)**:
  - **Default** (`make test` or `python3 -m pytest`): `tests/python/` + `tests/integration/` (same as `[tool.pytest.ini_options] testpaths` in `pyproject.toml`). CUDA/MPS/Ray/torch-only cases skip automatically when hardware or extras are missing.
  - **Fast E2E**: `make test-e2e-fast` — `tests/e2e` excluding `slow` and `benchmark` (avoids long `superoptimize()` runs).
  - **Full E2E**: `make test-e2e` — all E2E including superoptimize pipelines (can take many minutes on CPU).
  - **Extended Python**: `make test-py-extended` — default pytest plus fast E2E.
  - **C++**: `make test-cpp` after `make build` (ctest in `build/`).
  - **Coverage**: `make test-cov`.
- **Format**: `make format` (runs `black` and `isort`).

### Dependencies

Core Python dependencies are listed in `requirements.txt`. For development, install `[dev]` extras from `pyproject.toml`. PyTorch CPU-only is sufficient for development without GPU: `pip install torch --index-url https://download.pytorch.org/whl/cpu`.

Git submodules (`deps/cutlass`, `deps/json`, `deps/z3`) must be initialized: `git submodule update --init --recursive`.

### Key directories

| Directory | Purpose |
|-----------|---------|
| `python/yirage/` | Python package source |
| `src/` | C++ source (kernel, search, transpiler, backend) |
| `include/yirage/` | C++ headers |
| `tests/python/` | Python unit / API tests |
| `tests/integration/` | Integration tests (included in default `pytest`) |
| `tests/e2e/` | End-to-end (optional; use `make test-e2e` or `test-e2e-fast`) |
| `deps/` | Git submodules (cutlass, json, z3) |

---
> Source: [chenxingqiang/YiRage](https://github.com/chenxingqiang/YiRage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
