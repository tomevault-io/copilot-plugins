## h3c2p-bench

> - `just` is the entry point. `just` (default) builds and runs everything.

# H3 Benchmark Project

## Build system

- `just` is the entry point. `just` (default) builds and runs everything.
- H3 is built as a static library (`libh3.a`) via cmake, then the benchmark .c file is compiled separately against it with `cc`. The H3 submodule source trees are never modified (no patching CMakeLists.txt, no copying files in).
- Python is always run via `uv run`. Dependencies are declared inline in `bench.py` using PEP 723 script metadata. Never use bare `python`.

## Checkpoints

- Each `checkpoints/<name>/` directory is self-contained: an `h3/` submodule and a `benchmarkColorado.c`.
- `bench.py` auto-discovers checkpoints by scanning `checkpoints/` for directories containing both `h3/` and `benchmarkColorado.c`. No hardcoded list.
- To add a checkpoint: create the directory, add the submodule, write the benchmark .c file. No other files need to change.

## Benchmarking approach

- The C benchmark uses H3's `BENCHMARK` macro from `benchmark.h` which times only the body (not setup).
- `bench.py` runs each compiled binary multiple times and takes the minimum per resolution to filter system noise.
- The `RUNS` variable at the top of `bench.py` controls outer run count.
- Raw output from each run is saved to `results/raw/<checkpoint>/run_N.txt`.

## Code style

- Keep things simple. Prefer flat scripts over abstractions.
- The justfile should stay minimal. Shell logic uses shebang blocks when needed.
- The benchmark .c files are intentionally self-contained with duplicated setup code (`getColoradoCells`). This is preferred over a shared header.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajfriend)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ajfriend)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
