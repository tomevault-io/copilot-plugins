## deepseek-v4-flash-nvfp4-gb10-benchmark

> Guidance for AI coding agents (and humans) reproducing or extending this

# AGENTS.md — DeepSeek-V4-Flash Dual GB10 Benchmark

Guidance for AI coding agents (and humans) reproducing or extending this
benchmark. Read this fully before running anything.

## What this repo is
A reproducible benchmark of DeepSeek-V4-Flash (official FP8) served across
**2× NVIDIA DGX Spark (GB10, SM121 Blackwell)** with TP=2 + expert parallel +
MTP over RoCE, on the **fully native Blackwell tensor-core path** (DeepGEMM
FP8 block-scale + MXFP4 MoE, sparse MLA, Lightning Indexer). No Marlin, no
emulation, no CPU fallback.

## The one thing that matters most
**Throughput is determined by the vLLM build commit, not runtime flags.**
Use the prebuilt image (below) or build from pinned commit
`dda4668b59567416f86956cfe7bbc1eab371a61e` for `sm_121a`. That commit gates the
native Blackwell **family-120** fast path via `is_device_capability_family(120)`
(True for SM121 / capability 12.1) and includes the **rowwise paged-MQA logits
decode kernel**. Builds without it floor decode at ~5 tok/s instead of ~38.

## Fastest path: pull the prebuilt image
```bash
docker pull ghcr.io/r0b0tlab/vllm-dsv4-flash-gb10:cu130-sm121-arm64-dda4668b
```
The image is ARM64 / CUDA 13.0 / sm_121a, with torch 2.11.0+cu130 and the
pinned vLLM. You still supply the model weights (149GB, see below).

## Hardware assumptions (do not silently change)
- 2× DGX Spark (GB10, SM121), 128GB unified memory each.
- Direct QSFP56 200G link, RoCE/NCCL over CX-7, HCA `rocep1s0f0`.
- node0 = head (192.168.100.10), node1 = worker (192.168.100.11).
- Ethernet/control iface `enp1s0f0np0`.
If your IPs / interface / HCA names differ, update them in the launch env and
`docs/REPRODUCE.md`. Do not assume these are portable.

## Model
`deepseek-ai/DeepSeek-V4-Flash` — native FP8 E4M3, 128×128 block scale (UE8M0),
~149GB / 46 shards. Mount it read-only at `/mnt/model` on BOTH nodes. The repo
does NOT contain weights; download from Hugging Face.

## Rules for agents (hard constraints)
1. **Native-only.** Any result claimed must run the native DeepGEMM/sparse-MLA
   tensor-core path. Verify in the server log that DeepGEMM FP8/MXFP4 and the
   SM12x kernels are active and that there is NO Marlin/emulation/torch-einsum
   fallback. If a fallback is active, the run is invalid — do not report it.
2. **Quality gate before perf.** Run `scripts/quality_smoke.py`; require ≥4/5
   PASS before recording any throughput number.
3. **Median of ≥3.** Every reported tok/s is the median of ≥3 runs with fixed
   prompts and fixed decode length (256). Single runs are not results.
4. **Headline metric** = c=1 single-stream per-request decode tok/s. Aggregate
   throughput at c≥2 is secondary.
5. **Don't claim a number you didn't measure on this hardware class.** The
   public reference (~44 tok/s c=1) is from 2× DGX Spark; do not compare against
   different GPUs.

## Launch pitfalls (these WILL bite you)
- **Port 29501 EADDRINUSE**: zombie TCPStore processes survive a failed launch.
  `docker restart` BOTH containers before each launch attempt. `pkill` alone
  misses `multiprocessing.resource_tracker` children.
- **Launch order**: start the worker (node1) first, then the head (node0).
- **CUDA_HOME / torch libs**: the image's default env omits `CUDA_HOME` and
  torch/lib from `LD_LIBRARY_PATH`. The launch script exports them; keep that.
- **MTP spec config** must be
  `{"model":"/mnt/model","num_speculative_tokens":2,"method":"deepseek_mtp"}`.
  The bare string `"deepseek_mtp"` is rejected as an invalid HF repo id.
- **NCCL over RoCE**: set `NCCL_IB_DISABLE=0`, `NCCL_IB_HCA=rocep1s0f0`, and
  `{GLOO,TP,NCCL}_SOCKET_IFNAME=enp1s0f0np0`. If you skip the socket ifname,
  GLOO may pick a tailscale/100.x IP and break NCCL.
- **Building from source (only if not using the image)**: the runner stage's
  vLLM wheel install pulls CPU torch from PyPI and removes `libtorch_cuda.so`.
  Re-pin `torch==2.11.0+cu130` AFTER the wheel install (the Dockerfile here
  already does this; keep it).

## Two profiles (both in docs/REPRODUCE.md)
- Latency (best single-stream): `MAX_MODEL_LEN=200000 MAX_NUM_SEQS=2`
- Throughput (best aggregate): `MAX_MODEL_LEN=65536 MAX_NUM_SEQS=16`

## Verified results to match (median of 3, 256 decode tokens)
- Latency profile: c=1 38.4 tok/s, c=2 53.6 agg.
- Throughput profile: c=8 101.5 agg, c=16 144.6 agg.
If you get materially lower, you are almost certainly NOT on the native fast
path — check the build commit and the server log first.

## Don't
- Don't add Marlin/emulation and call it a result.
- Don't report single runs or numbers from a different GPU.
- Don't change the IPs/HCA/interface without saying so in your output.
- Don't claim the build succeeded without verifying `import vllm._C` works and
  `torch.version.cuda` is non-None inside the image.

---
> Source: [r0b0tlab/deepseek-v4-flash-nvfp4-gb10-benchmark](https://github.com/r0b0tlab/deepseek-v4-flash-nvfp4-gb10-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
