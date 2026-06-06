## ldpc-post-selection

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Python package for decoding quantum LDPC codes with cluster-based post-selection, implementing the approach from ["Efficient Post-Selection for General Quantum LDPC Codes"](https://arxiv.org/abs/2510.05795). The package provides confidence metrics for quantum error correction decoding based on error cluster statistics from BP+LSD decoders.

## Common Commands

```bash
# Install the main package (editable mode)
pip install -e .

# Install simulation suite (optional, for numerical analyses)
pip install -e ./simulations

# Run all tests
pytest

# Run specific test file
pytest tests/test_decoder.py

# Run a single test
pytest tests/test_decoder.py::TestSoftOutputsBpLsdDecoder::test_decode_single_sample -v
```

## Architecture

### Core Package (`src/ldpc_post_selection/`)

- **`base.py`**: `SoftOutputsDecoder` - Base class for decoders with soft outputs. Handles parity check matrix (H), observable matrix, and prior probabilities. Can be initialized from either matrices+priors or a `stim.Circuit`.

- **`bplsd_decoder.py`**: `SoftOutputsBpLsdDecoder` - Main decoder using BP+LSD (from `ldpc` package). Provides:
  - Standard decoding with cluster statistics (sizes, LLRs)
  - Sliding window decoding via `decode_sliding_window()` with caching for window structures and decoders
  - Logical gap proxy computation with methods: `None` (exhaustive), `'nearby'`, `'random'`, `'most-likely-first'`, `'weighted-random'`, `'most-likely-first-adaptive'`, `'weighted-random-adaptive'`

- **`logical_error_distribution.py`**: Utilities for collecting logical error distributions from simulations. Used with distribution-based gap proxy methods. See `docs/logical-gap-proxy-methods.md` for comprehensive method descriptions.

- **`matching_decoder.py`**: `SoftOutputsMatchingDecoder` - PyMatching-based decoder computing logical gap for confidence. Explores all logical classes (2^k for k observables) to find the gap between best and second-best predictions.

- **`cluster_tools.py`**: Utilities for cluster analysis including `compute_cluster_stats()`, `label_clusters()` (scipy-based), and `label_clusters_igraph()` (python-igraph based for performance).

- **`stim_tools.py`**: Stim circuit utilities - `dem_to_parity_check()` converts detector error models to sparse matrices.

### Simulations (`simulations/`)

Separate installable package for running numerical experiments. Contains simulation scripts for different code families:
- Surface codes, bivariate bicycle (BB) codes, hypergraph product (HGP) codes
- Both global and sliding window decoding variants
- Data collection and analysis pipelines in `analysis/`

## Key Dependencies

- `stim`: Quantum circuit simulation and detector error model generation
- `ldpc>=2.4.0`: BP+LSD decoder implementation
- `pymatching`: MWPM decoder
- `scipy.sparse`: Sparse matrix operations for parity check matrices
- `python-igraph`: Efficient graph operations for cluster labeling
- `numba`: JIT compilation for performance-critical paths

## Coding Patterns

- Parity check matrices are stored as `scipy.sparse.csc_matrix` with dtype `uint8`
- Decoders can be initialized from `stim.Circuit` (auto-extracts H, obs_matrix, priors) or explicit matrices
- Soft outputs are returned as dictionaries containing metrics like `pred_llr`, `detector_density`, `cluster_sizes`, `cluster_llrs`, `gap` or `gap_proxy`
- Sliding window decoding uses internal caching for decoder instances (keyed by matrix hash)

---
> Source: [seokhyung-lee/ldpc-post-selection](https://github.com/seokhyung-lee/ldpc-post-selection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
