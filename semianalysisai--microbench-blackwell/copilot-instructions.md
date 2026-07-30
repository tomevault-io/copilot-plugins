## microbench-blackwell

> Measures LDGSTS (`cp.async` / `__pipeline_memcpy_async`) throughput on Blackwell (B200).

# LDGSTS Throughput Microbenchmark

## What This Benchmarks
Measures LDGSTS (`cp.async` / `__pipeline_memcpy_async`) throughput on Blackwell (B200).
Uses `ncu` hardware counters for precise per-instruction metrics.

## FlashInfer Kernel Configs (LDGSTS Usage)

### Blackwell MHA Kernel (sm100_fmha_fwd_kernel_tma_warpspecialized)
- **Does NOT use LDGSTS** — uses TMA for all global→shared loads
- 512 threads (16 warps), 1 CTA/SM, cluster 1×1×1
- TMA works because KV cache is contiguous

### Blackwell MLA Kernel (sm100_fmha_mla_tma_warpspecialized)
- **Uses LDGSTS** (`cp.async.cg.shared.global ... 16`) because paged KV cache requires per-thread address computation (page table indirection breaks TMA's regular stride assumption)
- Key files:
  - `include/flashinfer/attention/blackwell/kernel/sm100_fmha_mla_tma_warpspecialized.hpp`
  - `include/flashinfer/attention/blackwell/collective/sm100_fmha_load_cpasync_warpspecialized.hpp`
  - `include/flashinfer/attention/cutlass_mla.cuh` (instantiation)

#### MLA Kernel Config
- **256 threads (8 warps)** per CTA, 2SM cluster (ClusterShape = 2×1×1)
- **2 load warps** (64 threads) doing LDGSTS
- **12 pipeline stages** for BF16 (`StagesQK = 24 / sizeof(Element)`)
- **16-byte loads** (`cp.async.cg.shared.global ... 16` via `uint128_t`)
- **1 CTA per SM** (paired across 2 SMs for 2SM UMMA)
- 2SM cluster is for cooperative UMMA (tensor core spans both SMs), LDGSTS is per-CTA

#### MLA Warp Roles (kIsCpAsync path, decoded from 0x4221'3333)
```
Warp 0-3: Compute (softmax/rescale)
Warp 4:   MMA (UMMA tensor core)
Warp 5-6: Load (LDGSTS)
Warp 7:   LoadPageTable
```

#### MLA Tile Shapes
- TileShape = (128, 128, (512, 64)) — H=128 Q tokens, S=128 KV tokens, D=(latent=512, rope=64)
- QK tile: (128, 128, 64) — rope dim iterated over latent+rope
- PV tile: (128, 256, 32)

#### Why Only 2 Load Warps
- Kernel is softmax-bottlenecked (scalar CUDA core math: exp2, row max/sum)
- 4 warps needed for softmax to keep up with MMA
- B200 roofline crossover ~280 FLOPs/byte; MLA attention is ~130 FLOPs/byte (memory-bound)
- 12 pipeline stages compensate for low load warp count via latency hiding
- Potential optimization: 12 warps total with 4 load warps (register pressure tradeoff)

### Pre-Blackwell MHA Kernels (SM80/SM90, uses LDGSTS)
- SM80 MHA (`csrc/xqa/mha.cu`): 256 threads (8 warps), 4 IO warps (128 threads) loading, 2 stages
- SM90 MHA (`csrc/xqa/mha_sm90.cu`): 384 threads (12 warps), 4 IO warps (128 threads) loading, 2 stages, 2-3 CTAs/SM
- Load size: 16 bytes (`ldgsts::copyAsync<16>`)
- Load wrapper: `csrc/xqa/ldgsts.cuh`

## Benchmark Results Summary

### Pre-Blackwell MHA Configs (ldgsts_tput_results.csv)
- Sweep: CTAs 1-4, stages 1/2/4, threads 64/128/256, load float/float2/float4
- Peak throughput: ~6.8 TB/s with float4 at 48+ KiB bytes-in-flight
- Saturation begins ~16-32 KiB bytes-in-flight per SM

### MLA Configs (ldgsts_mla_results.csv)
- Sweep: 1 CTA, stages 4/8/12/16, threads 64/128/256, float4
- MLA kernel config (2 warps, 12 stages): ~2.26 TB/s, 2.25 LDGSTS/cyc
- Throughput scales ~linearly with load warp count
- Stages saturate around 8-12; diminishing returns past 8

## Build & Run
```bash
# Single config
make ldgsts_tput.out CTAS_PER_SM=1 NUM_STAGES=12 THREADS_PER_BLOCK=64 LOAD_T=float4

# Profile
make bench

# Full sweep (pre-Blackwell MHA defaults)
python3 benchmark.py -o results.csv -v --overwrite

# MLA-like sweep
python3 benchmark.py --ctas 1 --threads 64,128,256 --stages 4,8,12,16 --load-types float4 -o mla_results.csv -v

# Plot
.venv/bin/python plot_ldgsts_tput.py   # MHA results
.venv/bin/python plot_ldgsts_mla.py    # MLA results
```

---
> Source: [SemiAnalysisAI/microbench-blackwell](https://github.com/SemiAnalysisAI/microbench-blackwell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
