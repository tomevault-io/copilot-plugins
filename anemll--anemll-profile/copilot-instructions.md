## anemll-profile

> Use `anemll-profile` to profile and optimize Apple Neural Engine Core ML models.

# anemll-profile Agent Guide

Use `anemll-profile` to profile and optimize Apple Neural Engine Core ML models.

## Install

If Homebrew is available:

```bash
brew install anemll/tap/anemll-profile
```

If building from source:

```bash
xcode-select --install
make
sudo make install
```

Verify the install:

```bash
anemll-profile --version
```

## Dependencies

Required:

- macOS 14+ with the system `CoreML` and `Foundation` frameworks
- Xcode Command Line Tools for `xcrun clang`, `make`, and SDK headers

Recommended for agent workflows:

- `jq` for reading `-j` JSON reports

There are no Python or pip dependencies for `anemll-profile` itself.

## Preferred Mode

Prefer JSON output for agent workflows:

```bash
anemll-profile -j report.json model.mlmodelc
anemll-profile -j report.json model.mlpackage
```

The text report is good for a quick human scan, but JSON is the source of truth for automation.

## What To Read First

Focus on these JSON sections:

- `summary`
- `measured_prediction`
- `graph_interruptions`
- `fallback_ops`
- `op_type_breakdown`
- `top_operations`

## Fast Workflow

1. Run the profiler with `-j`.
2. Check `graph_interruptions`.
3. If interruptions are `0`, shift attention to `fallback_ops`, `op_type_breakdown`, and `top_operations`.
4. If interruptions are nonzero, rank fixes by `estimated_total_tax_ms` and the top interruption islands.
5. Use `measured_prediction` to compare real impact after model changes.

## How To Interpret Results

- `graph_interruptions`: Non-ANE islands that break continuous ANE execution. The detour can be CPU or GPU.
- `fallback_ops`: The best place to see why specific ops did not stay on ANE.
- `op_type_breakdown`: Best high-level view of memory-bound vs compute-bound hotspots.
- `top_operations`: Best place to find repeated expensive state updates, slices, LUT expansion, or large memory moves.

## Common Optimization Signals

- `Unsupported tensor data type: int32`: Try to reduce or isolate int32 control and indexing work.
- Repeated `slice_update`, `slice_by_index`, or `read_state`: Investigate KV-cache layout and state update strategy.
- Large `constexpr_lut_to_dense` cost: LUT expansion may be dominating memory bandwidth.
- Few CPU fallback ops with `0` interruptions: The model is mostly staying on ANE; focus on memory traffic before chasing interruption fixes.

## Model Directory Tip

If a folder contains multiple `.mlmodelc` or `.mlpackage` bundles, profile the important submodels individually, such as:

- embeddings
- prefill
- decode or FFN body
- lm_head

Do not assume a parent directory is directly profileable unless it contains a single model bundle.

---
> Source: [Anemll/anemll-profile](https://github.com/Anemll/anemll-profile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
