## mlp-megakernel

> Context file for AI agents working on this repo.

# AGENTS.md — MLP Megakernel

Context file for AI agents working on this repo.

## What this repo is

Triton megakernels that fuse multi-layer MLPs with Softplus activations into
single GPU kernel launches. The core idea: **register-tiled recompute fusion**
— all GEMMs and activations happen in one kernel, with intermediate activations
living entirely in registers (never written to global memory).

Originated from [KernelBench](https://github.com/RightNow-AI/autokernel)
Problem L2_P900 (fused_mlp_softplus_3layer). This repo contains only the
megakernel code and reports, not the KernelBench infrastructure.

## Branches

| Branch | Purpose |
|--------|---------|
| `main` | Inference-only 3-layer megakernel |
| `feature/weight-norm` | Full training (fwd+bwd) for 3-layer and 5-layer, with weight normalization, AMP support, autograd integration, benchmarks, and detailed reports |

## Architecture

```
3-layer: out = softplus(softplus(x @ W1) @ W2) @ W3
5-layer: out = softplus(softplus(softplus(softplus(x @ W1) @ W2) @ W3) @ W4) @ W5
```

Default dimensions: D=128, H=128 (all hidden layers). Batch sizes from 256 to
65536. FP16 with FP32 accumulation (Tensor Cores).

## File map (main branch)

| File | What it does |
|------|--------------|
| `kernel.py` | The megakernel. Contains `_fused_mlp_recompute` (Triton JIT), `_pick_config` (runtime heuristic), `_fused_mlp_fwd` (launch wrapper), `Model` (PyTorch reference), `ModelNew` (fused version) |
| `mlp_fused.md` | Detailed optimization report covering the forward kernel design, backward kernel, heuristics, and benchmark results |
| `README.md` | Project overview and quick start |

## How the kernel works

### Forward (`_fused_mlp_recompute`)

Persistent kernel: launches `min(NUM_SMs, num_tiles)` programs that iterate
tiles round-robin. Each tile computes a `[BM, BN3]` output block via nested
loops:

```
for k3 (over N2, step BK3):      # h2 columns
  for k2 (over N1, step BK2):    # h1 columns
    for k1 (over K1, step BK1):  # input columns
      h1_chunk += X_tile @ W1_tile
    h1_chunk = softplus(h1_chunk)     # in-register
    h2_chunk += h1_chunk @ W2_tile
  h2_chunk = softplus(h2_chunk)       # in-register
  out_acc += h2_chunk @ W3_tile
store(OUT, out_acc)
```

- All `tl.dot` accumulate in fp32; cast to fp16 only at tile boundaries
- Softplus: `where(x > 20, x, log(exp(x) + 1))` (numerically stable)
- GROUP_M=16 swizzle for L2 cache locality on weight loads
- BK values match hidden dims (128) to avoid recomputation redundancy

### Config heuristic (`_pick_config`)

Selects `BM` and `warps` at runtime based on batch size:

- Computes `target = M * num_output_tiles / NUM_SMs`
- Small target (≤24) → BM=16, warps=4 (high occupancy)
- Medium target (≤48) → BM=32, warps=8
- Large target → BM=64, warps=8
- FP32 mode uses warps=4 throughout (register pressure)

No warmup benchmarking. Pure arithmetic heuristic.

## Key constraints & gotchas

1. **Requires Tensor Core GPU** (sm_80+). The kernel uses `tl.dot` which maps
   to HMMA instructions.
2. **K1, N1, N2 must be `tl.constexpr`** — they control `static_range` loop
   bounds. N3 is dynamic (masked loads).
3. **BK values must divide hidden dims exactly** — no remainder handling inside
   the nested loops. With D=H=128 and BK=64 or 128 this is always satisfied.
4. **FP16 only on main branch** — the kernel hardcodes `.to(tl.float16)` at
   tile boundaries. The `feature/weight-norm` branch adds FP32 dtype support
   via `INPUT_DTYPE` constexpr.
5. **`NUM_SMS` is computed at import time** — `torch.cuda.get_device_properties(0)`.
   This means `kernel.py` requires a CUDA device at import.

## Running

```bash
# Quick correctness test
python -c "
import torch
from kernel import Model, ModelNew
x = torch.randn(4096, 128, dtype=torch.float16, device='cuda') * 0.02
w1 = torch.randn(128, 128, dtype=torch.float16, device='cuda') * 0.02
w2 = torch.randn(128, 128, dtype=torch.float16, device='cuda') * 0.02
w3 = torch.randn(128, 128, dtype=torch.float16, device='cuda') * 0.02
ref = Model()(x, w1, w2, w3)
out = ModelNew()(x, w1, w2, w3)
print('max diff:', (ref - out).abs().max().item())
print('cosine sim:', torch.nn.functional.cosine_similarity(ref.flatten(), out.flatten(), dim=0).item())
"
```

## Performance (A100, D=H=128)

The fused megakernel achieves ~1.12x speedup over PyTorch eager at M=4096.
Speedup increases with batch size due to better SM utilization. See
`mlp_fused.md` for full benchmark tables.

## Dependencies

- PyTorch >= 2.0
- Triton >= 2.1
- CUDA GPU with Tensor Core support (sm_80+)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Marcelo5444)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Marcelo5444)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
