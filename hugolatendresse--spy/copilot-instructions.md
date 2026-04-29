## spy

> Project context for DuckDB/RPT+ TieredHashCache fork


# Project Details

This project implements a fork of DuckDB / RPT+ with a TieredHashCache (THC). See `tiered_hash_cache.hpp` and `join_hashtable.cpp` for the details.

RPT+ implements a forward pass with Bloomfilters to do information passing.

The idea of the THC is that the hot portion of the Hash Table built on the build-side of Hash Joins can sometimes fit in the L3 cache, even if the entire Hash Table is much bigger. If we do RPT+ forward pass only, the build-side hash table will contain tuples we don't need, but since everything fits in L3 cache, it is faster to have a THC than to use RPT+ backward pass.

TieredHashCache copies the accessed rows to a new buffer, and we can hope that this buffer will stay in L3 and that the CPU cache will contain mostly hot build side entries instead of containing build side entries that will never be accessed.

THC tries to adaptively see if it's bringing any benefits. It sometimes stops the COLLECT phase for example. See details in the implementation.

## Build Command

```bash
GEN=ninja BUILD_BENCHMARK=1 BUILD_TPCH=1 BUILD_TPCDS=1 BUILD_HTTPFS=1 CORE_EXTENSIONS='tpch' EXTRA_CMAKE_VARIABLES='-DCMAKE_CXX_FLAGS=-DExactLeftDeep' make release -j $(nproc)
```

## Workflow

Write a report as you go of the changes you implement and the optimizations you try/keep and all effects on the average run time results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hugolatendresse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
