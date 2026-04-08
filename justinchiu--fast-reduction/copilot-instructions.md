## fast-reduction

> Implement a **fused linear + cross-entropy + entropy** kernel for H100 (Hopper, sm90) that hits

# fast-reduction

## Goal

Implement a **fused linear + cross-entropy + entropy** kernel for H100 (Hopper, sm90) that hits
>= 90 % of H100 HBM3 peak throughput (3.35 TB/s).

## Hardware

8 x H100 80 GB (Hopper, sm90, HBM3).

## Performance

See README.md for up-to-date forward and forward+backward results tables.

**CuTe reduction kernels alone** (no matmul, B=4096, V=128256, fp32):

| Kernel | Time | BW | % of H100 peak |
|--------|------|----|-----------------|
| CE only | 0.70 ms | 2987 GB/s | 89% |
| entropy only | 0.77 ms | 2740 GB/s | 82% |
| joint CE+entropy | 0.76 ms | 2770 GB/s | 83% |

The reduction kernels are near speed of light. The bottleneck in levels 1-4 is the
`torch.mm` matmul (~34 TFLOP per call at these sizes). Level 5 eliminates the logits
HBM round-trip entirely by computing reductions in the GEMM epilogue.

## Architecture: what's fused and what's not

Levels 1-4 all do **chunked matmul** (`torch.mm` via cuBLAS) followed by a
separate **CuTe DSL reduction kernel**. The logits `[chunk, V]` buffer is
reused each iteration, so peak memory is O(chunk_size * V) instead of O(B * V).
But the logits still go through HBM between the matmul and the reduction.

Level 5 fuses the reduction into the GEMM epilogue so logits stay in
registers/SMEM and never touch HBM, using quack's extensible epilogue
infrastructure (`gemm_sm90.py` with `epi_visit_subtile()` hooks).

Level 5M (megakernel) extends this for training: the epilogue writes both
partial reduction stats AND fp32 logits to GMEM, then a fused CuTe dlogits
kernel computes dlogits in one pass. This eliminates the cuBLAS logits
recompute and gives fp32-precision gradients from the WGMMA accumulator.

## Key files

```
fast_reduction/
  __init__.py
  baseline.py              pure-PyTorch reference (unfused + chunked + backward)
  kernel.py                chunked matmul + CuTe reduction (levels 3 & 4) + backward + autograd
  cute_cross_entropy.py    CuTe DSL kernels: CE-only, entropy-only, joint CE+entropy
  cute_dlogits.py          fused CuTe dlogits kernel (level 5M backward)
  reduce.py                reduction primitives (thread → warp → block → cluster)
  cute_utils.py             low-level PTX: DSMEM, f32↔i64 packing, pointer arithmetic
  gemm_ce_entropy_epilogue.py       Level 5 GEMM epilogue mixin (single-pass)
  gemm_ce_entropy_epilogue_v2.py    Level 5.2 GEMM epilogue mixin (two-pass)
  gemm_ce_entropy_bwd_epilogue.py   Level 5M GEMM epilogue (partials + logits output)
  gemm_ce_entropy_finalize.py       Level 5 finalization kernel
  gemm_kernel.py                    Level 5 driver + megakernel + backward + autograd

benchmarks/
  bench_linear_xent_entropy.py   forward-only: wall-clock, peak mem, model BW, accuracy
  bench_fwd_bwd.py               forward + backward: time, peak mem, gradient accuracy
  profile.sh                     Nsight Compute profiling script

tests/
  test_ce_impls.py         forward correctness tests vs PyTorch reference
  test_backward.py         backward gradient correctness tests vs fp32 reference
  test_entropy_large_v.py  large-V entropy accuracy tests (cluster reduction)

docs/
  memory_bound_kernels.md  reference blogpost (Guo, Zadouri, Dao)
```

## Testing

```bash
uv run pytest
uv run pytest tests/test_ce_impls.py -v
```

## Benchmarking

```bash
# Forward-only benchmark
uv run python benchmarks/bench_linear_xent_entropy.py
uv run python benchmarks/bench_linear_xent_entropy.py --B 32768 --V 128256 --H 4096
uv run python benchmarks/bench_linear_xent_entropy.py --no-accuracy   # speed/memory only

# Forward + backward benchmark
uv run python benchmarks/bench_fwd_bwd.py
uv run python benchmarks/bench_fwd_bwd.py --B 4096 --V 128256 --H 4096
```

## API

```python
from fast_reduction.kernel import fused_linear_xent_entropy, separate_linear_xent_entropy

# Level 4: joint CE+entropy (one CuTe kernel per chunk) — forward only
ce_loss, entropy, log_probs = fused_linear_xent_entropy(
    hidden_states,  # [B, H]  bf16 or fp32
    weight,         # [V, H]
    target,         # [B]     torch.long
    bias=None,      # [V]     optional
    chunk_size=4096,
)

# Level 3: separate CE + entropy (two CuTe kernels per chunk) — forward only
ce_loss, entropy, log_probs = separate_linear_xent_entropy(
    hidden_states, weight, target, chunk_size=4096,
)
```

### Training API (fast forward + backward)

```python
from fast_reduction.gemm_kernel import gemm_megakernel_fast

# Level 5M: GEMM epilogue + fused CuTe dlogits (recommended for training)
hidden.requires_grad_(True)
weight.requires_grad_(True)
loss, ce_loss, entropy = gemm_megakernel_fast(
    hidden, weight, target, chunk_size=4096,
    ce_weight=1.0, ent_weight=-1.0,
)
loss.backward()  # hidden.grad, weight.grad populated
# ce_loss, entropy are detached (for logging)

# Level 4F: CuTe joint fast (cuBLAS matmul + CuTe reduction)
from fast_reduction.kernel import fused_linear_xent_entropy_fast
loss, ce_loss, entropy = fused_linear_xent_entropy_fast(
    hidden, weight, target, chunk_size=4096,
)
```

The fast variants pre-compute gradients during the forward pass
(liger/quack pattern), so backward just scales by the upstream scalar.

Ground truth: `fast_reduction.baseline_linear_xent_entropy` (same signature minus chunk_size).
Ground truth backward: `fast_reduction.baseline_linear_xent_entropy_backward`.

## Dependencies

```toml
[project]
dependencies = [
    "nvidia-cutlass-dsl>=4.4.1",
    "apache-tvm-ffi",
    "torch>=2.5",
]
```

## GPU Usage

This machine has 8 x H100 GPUs. Always use an idle GPU for experiments to avoid
interfering with other work.

### Check GPU utilization

```bash
# Quick check: shows index, utilization %, memory used/total
nvidia-smi --query-gpu=index,utilization.gpu,memory.used,memory.total --format=csv,noheader

# Full dashboard
nvidia-smi
```

### Run on a specific unused GPU

```bash
# Run on GPU 1
CUDA_VISIBLE_DEVICES=1 uv run python benchmarks/bench_linear_xent_entropy.py

# Run on GPU 3
CUDA_VISIBLE_DEVICES=3 uv run pytest tests/test_ce_impls.py -v
```

Always check `nvidia-smi` first, pick a GPU with 0% utilization and 0 MB used,
then prefix your command with `CUDA_VISIBLE_DEVICES=<gpu_id>`.

## Commit Contract

Every commit must include these sections in the commit message.

### 1) High-Level Plan

State:
- what problem is being solved,
- what approach is being taken,
- what files/components are affected,
- what risks or tradeoffs are expected.

### 2) Exploration / What Didn't Work

Document the journey, not just the destination:
- what alternatives were tried and why they failed,
- dead ends, surprising findings, performance cliffs,
- how the final approach was arrived at and why it won.

This is the most valuable part of the commit — future readers need to understand
*why* the code looks the way it does, not just *what* it does.

### 3) Red-Green TDD Evidence

Show the test-driven sequence:
- **RED**: the new/updated test that fails before the fix (include command and failure signal),
- **GREEN**: the same test passing after the fix,
- **REGRESSION CHECK**: relevant suite passing (`tests/test_ce_impls.py`, `tests/test_backward.py`, or full `pytest` as appropriate).

If true RED cannot be demonstrated (e.g. infrastructure-only change), explicitly state why.

### 4) Numerical Results → README.md

For kernel/math/performance changes, report before/after numbers with command used and exact shape/config:
- `B`, `H`, `V`, `chunk`, dtype, GPU,
- runtime (ms),
- peak memory (GB),
- numerical error metrics (CE MAE, entropy MAE, and gradient MAE when applicable).

If a change is not expected to affect numerics/perf, include "Numerical impact: N/A" and still provide test evidence.

**Update README.md** with the final results whenever performance or accuracy tables change.
The README is the single source of truth for published numbers.

### Suggested Commit Template

```text
<scope>: <short summary>

Plan:
- ...

Exploration:
- Tried X, but Y because Z
- Found that A was the bottleneck, not B as expected
- Final approach: ...

TDD:
- RED: <command> -> <failing assertion/error>
- GREEN: <command> -> <pass result>
- REGRESSION: <command> -> <pass result>

Numerical results:
- Config: B=..., H=..., V=..., chunk=..., dtype=..., GPU=...
- Before: time=... ms, peak=... GB, CE_MAE=..., ENT_MAE=..., dH_MAE=...
- After:  time=... ms, peak=... GB, CE_MAE=..., ENT_MAE=..., dH_MAE=...
```

## Reference

- Quack repo (Dao-AILab/quack) — CuTe DSL cross-entropy, softmax, and GEMM kernels. Clone at ~/quack.
- Quack GEMM epilogue framework: `quack/gemm_sm90.py` (extensible mixin with `epi_visit_subtile()`)
- Quack epilogue examples: `gemm_act.py` (activation fusion), `gemm_dact.py` (backward activation)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justinchiu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/justinchiu)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
