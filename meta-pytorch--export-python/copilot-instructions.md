## export-python

> `torch_export_python` exports PyTorch models as readable, editable Python code. It covers AOTAutograd (forward+backward autograd functions) and Inductor (fused Triton kernels).

# CLAUDE.md

## Overview

`torch_export_python` exports PyTorch models as readable, editable Python code. It covers AOTAutograd (forward+backward autograd functions) and Inductor (fused Triton kernels).

## Running Tests

This project uses pytest directly (no Buck targets):

```bash
# Run all tests from the project root
pytest tests/

# Run a specific test
pytest tests/test_export.py -k test_name

# Some tests require a GPU and are skipped automatically on CPU-only machines
```

### Snapshot Testing

Tests compare generated code against snapshot files in `tests/snapshots/`. To update snapshots after intentional changes:

```bash
EXPECTTEST_ACCEPT=1 pytest tests/
```

## Linting

```bash
arc f     # autoformat
arc lint  # check lint
```

---
> Source: [meta-pytorch/export-python](https://github.com/meta-pytorch/export-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
