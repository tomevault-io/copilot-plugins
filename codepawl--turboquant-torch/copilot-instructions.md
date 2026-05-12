## turboquant-torch

> PyTorch-based vector quantization library. Unofficial implementation of TurboQuant (ICLR 2026).

# TurboQuant-Torch

PyTorch-based vector quantization library. Unofficial implementation of TurboQuant (ICLR 2026).

- Paper: https://arxiv.org/abs/2504.19874
- Repo: https://github.com/codepawl/turboquant-torch

## Branching Strategy

- **main** — Production branch. Only merge stable, tested code here.
- **staging** — Prototype/pre-production branch. Use for experimental features and integration testing before promoting to main.

Feature branches should be created from `staging` and merged back into `staging` via PR. Once validated, `staging` is merged into `main`.

## Versioning

- Two version sources kept in sync: **git tags** (for `setuptools-scm` builds) and `turboquant/__init__.py __version__` (for runtime).
- Tag format: `v{MAJOR}.{MINOR}.{PATCH}` (SemVer)
  - PATCH: bug fixes, docs, CI improvements
  - MINOR: new features, no breaking changes
  - MAJOR: stable API or breaking changes
- **Use `./scripts/version.sh`** to bump — it updates `__init__.py`, commits, and tags in one step:
  ```bash
  ./scripts/version.sh              # show current version
  ./scripts/version.sh patch        # 0.2.1 → 0.2.2
  ./scripts/version.sh minor        # 0.2.1 → 0.3.0
  ./scripts/version.sh major        # 0.2.1 → 1.0.0
  ./scripts/version.sh 1.2.3        # set exact version
  ./scripts/version.sh patch --dry  # preview without changing anything
  ```
- Release flow (from staging):
  ```bash
  ./scripts/version.sh patch
  git push origin staging            # wait for CI green
  git checkout main && git merge staging
  git push origin main --tags        # triggers build + TestPyPI + PyPI
  ```

## Project Structure

```
turboquant/
├── __init__.py          # Public API + __version__
├── core.py              # TurboQuant two-stage quantizer (MSE + QJL)
├── mse_quantizer.py     # Stage 1: MSE-optimal scalar quantization
├── qjl.py               # Stage 2: QJL 1-bit correction for unbiased IP
├── hadamard.py          # Fast Walsh-Hadamard Transform + random rotation
├── codebook.py          # Lloyd-Max codebook computation (Gaussian + Beta)
├── codebook_data.py     # Precomputed Beta codebooks (generated, do not edit)
├── kv_cache.py          # KV cache compression (sliding window, GQA, Pre-RoPE)
├── outlier.py           # Outlier channel detection and routing
├── adaptive.py          # Adaptive per-layer bit allocation
├── compat.py            # Model architecture compatibility detection
├── rope.py              # RoPE utilities for Pre-RoPE key quantization
└── vector_search.py     # Approximate nearest neighbor search

tests/                   # pytest tests (mirrors turboquant/ modules)
examples/                # Practical example scripts (quickstart, long context, adaptive)
├── demo/                # Interactive rich demo (demo_live.py)
benchmarks/              # Benchmark scripts + result data + card generation
scripts/
├── version.sh           # Version bumping tool
└── precompute_codebooks.py  # Regenerate codebook_data.py
assets/                  # PNG cards for README / X posts
```

## Key Commands

```bash
# Lint & format
ruff check turboquant/ tests/ examples/
ruff format turboquant/ tests/ examples/
uv run mypy turboquant/

# Tests
pytest tests/ -v                    # all tests
pytest tests/test_kv_cache.py -v    # single module
pytest tests/ -k "sliding"          # by keyword

# Benchmarks
python benchmarks/run_benchmarks.py           # core KV cache benchmarks
python benchmarks/bench_sliding_window.py     # sliding window sweep
python benchmarks/bench_gqa.py                # GQA error amplification
python benchmarks/generate_cards.py           # regenerate all PNG cards

# Codebook precomputation (only after changing codebook.py)
python scripts/precompute_codebooks.py        # regenerates codebook_data.py
```

## CI

- Runs on GitHub Actions with **uv** for dependency management
- Pipeline: Security Scan → Lint & Type Check → Test (Python 3.10–3.14) → Build Package
- Publish workflow triggers on `v*.*.*` tags (TestPyPI + PyPI)
- Triggered on push/PR to `main` and `staging`

## Code Conventions

- Python 3.10+ (target-version in ruff and mypy)
- Line length: 100 (ruff)
- Ruff rules: E, F, I, W, UP, B, SIM, S, C4, RET, PTH
- `S101` (assert) and `S311` (random) suppressed in tests/examples
- Type hints required on public APIs; `from __future__ import annotations` for `X | None` syntax
- Docstrings: Google style with Args/Returns sections
- Tests: class-based grouping (`class TestFeatureName`), descriptive docstrings
- NamedTuple for data containers (CompressedKV, TurboQuantOutput, etc.)

## Rules for Claude Code

1. **Merge before moving on.** Before starting any new branch, check for unmerged feature/fix branches (`git branch --list "fix/*" "feat/*" "feature/*"`). Verify each passes lint + typecheck + pytest. If clean, create PR and squash merge to staging. If broken, report and stop. Always start new work from latest staging.

## Architecture Notes

- **TurboQuant** (core.py): two-stage quantizer. Stage 1 = MSE-optimal scalar quantization via Lloyd-Max codebooks. Stage 2 = QJL 1-bit correction on the residual for unbiased inner product estimation.
- **KV Cache** (kv_cache.py): wraps two TurboQuant instances (keys=unbiased, values=biased). Features: sliding window (residual_length), asymmetric key/value bit widths, GQA-aware auto-tuning (for_gqa factory), Pre-RoPE key quantization.
- **Codebook** (codebook.py): exact Beta(d/2, d/2) codebooks for dim < 256 (precomputed in codebook_data.py), Gaussian approximation for dim ≥ 256. Threshold was raised from 64→256 in v0.2.2.
- **Hadamard** (hadamard.py): randomized rotation before quantization to make coordinates approximately Beta-distributed. Pads to next power of 2.

---
> Source: [codepawl/turboquant-torch](https://github.com/codepawl/turboquant-torch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
