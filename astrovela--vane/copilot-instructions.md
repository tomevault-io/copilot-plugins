## vane

> Follow the [Development Guide](https://vane.astrovela.ai/docs/data/contributing/development) for the development workflow.

# AI Agent Guidelines

Follow the [Development Guide](https://vane.astrovela.ai/docs/data/contributing/development) for the development workflow.

## Build

Do not use an editable install. Python-only changes do not require a native rebuild. After changing C++, reinstall using the incremental build directory:

```bash
export SKBUILD_BUILD_DIR="$PWD/build/python-release"
export SKBUILD_CMAKE_BUILD_TYPE=Release
uv pip install . --no-build-isolation
```

## Formatting

```bash
scripts/format root --changed
scripts/format submodule --changed
scripts/format workspace --changed
```

Use `root` for the main repository and `submodule` for `external/duckdb`. Use `workspace` only when both contain changes.

## Tests

Run the tests affected by the change first, then run the Vane base test suite:

```bash
python -m pytest tests/fast/test_udf_process.py
scripts/run_release_tests.sh
```

To run the complete fast Python test suite:

```bash
python -m pytest tests/fast
```

---
> Source: [AstroVela/vane](https://github.com/AstroVela/vane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
