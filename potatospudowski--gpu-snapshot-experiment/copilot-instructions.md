## gpu-snapshot-experiment

> Demonstrating three cumulative cold-start optimizations for GPU inference containers, for a LinkedIn post series + side-by-side video comparison.

# CLAUDE.md

## Project: GPU Cold Boot Optimization PoC

Demonstrating three cumulative cold-start optimizations for GPU inference containers, for a LinkedIn post series + side-by-side video comparison.

**Model**: Qwen/Qwen2.5-7B-Instruct (14.2 GiB, BF16)
**GPU**: NVIDIA L40S (48GB vRAM)
**Target audience**: DevOps/MLOps folks running Kubernetes GPU workloads

---

## Test VM

- **Instance**: `i-03c6ab74b910b2cbf` (g6e.xlarge, L40S)
- **IP**: `52.23.175.105`
- **Region**: us-east-1
- **SSH**: `ssh -i ~/.ssh/gpu-workmachine.pem ubuntu@52.23.175.105`
- **OS**: Ubuntu 22.04.5 LTS, kernel 6.8.0-1055-aws
- **NVIDIA driver**: 580.159.03
- **CUDA**: 12.9
- **Docker**: 29.5.1 (experimental mode enabled)
- **containerd**: 2.2.3
- **CRIU**: 4.0 (built from source, installed at `/usr/local/bin/criu.real`)
- **cuda-checkpoint**: 580.159.03 (from NVIDIA/cuda-checkpoint repo)
- **CRIU CUDA plugin**: `/usr/lib/criu/cuda_plugin.so` (built with CRIU 4.0)
- **Docker data-root**: `/opt/dlami/nvme/docker` (moved to local NVMe for speed)
- **NVMe instance store**: `/opt/dlami/nvme` (229GB, ~922 MB/s sequential read)
- **EBS root**: `/dev/root` 1TB gp3 (~111-167 MB/s sequential read)
- **ECR repo**: `497919589513.dkr.ecr.us-east-1.amazonaws.com/gpu-cold-boot`
- **HF model cache**: `/home/ubuntu/.cache/huggingface/hub/` (7B model pre-downloaded)

---

## What Worked

### 1. Bare-process CRIU + cuda-checkpoint (the gold standard)

**Flow**: Run Python process on host → load model → warmup → CRIU dump with CUDA plugin → CRIU restore

- CRIU 4.0 with CUDA plugin handles GPU state automatically (lock → checkpoint → dump → restore → GPU restore → unlock)
- **Checkpoint**: ~82s for 17GB snapshot (process + 15GB GPU memory)
- **Restore from NVMe (cold)**: ~47s
- **Restore from page cache (warm)**: ~11.5s
- **Time from process-wake to first inference**: 1.5s consistently
- Snapshot stored at `/opt/dlami/nvme/snapshot-7b/`

### 2. `runc checkpoint` on Docker containers

**Flow**: `docker run --gpus all` → model loads inside container → `sudo runc --root /run/docker/runtime-runc/moby checkpoint --tcp-established --image-path <dir> <container-id>`

- Works! Produces valid 17GB checkpoint including GPU state
- Took ~82s for 7B model, ~10s for 1.5B model
- The CRIU CUDA plugin handles GPU lock/checkpoint/restore transparently
- Container exits after checkpoint (expected behavior)

### 3. `docker start --checkpoint` restores containers with GPU state

- After putting the runc checkpoint into `/var/lib/docker/containers/<id>/checkpoints/<name>/`, `docker start --checkpoint <name> <container>` successfully restores
- GPU memory comes back (15.2GB on GPU, python process alive)
- CUDA functional after restore (`docker exec` can run torch operations)

### 4. CRIU 4.0 vs 3.16

- CRIU 3.16 (Ubuntu repo) **segfaults** on torch process restore. Unusable.
- CRIU 4.0 (built from source) works perfectly. Required for this project.
- Build deps: `libprotobuf-dev libprotobuf-c-dev protobuf-c-compiler protobuf-compiler python3-protobuf libcap-dev libnl-3-dev libnet1-dev libaio-dev libgnutls28-dev`
- Source: `/tmp/criu-4.0/` on the VM

### 5. CRIU wrapper for timeout fix

Docker/runc hardcodes no timeout for CRIU, and the CUDA plugin's default CRIU timeout is 10s — too short for GPU checkpoint. Fixed with wrapper at `/usr/local/bin/criu`:
```bash
#!/bin/bash
exec /usr/local/bin/criu.real --timeout 300 "$@"
```
This lets `docker checkpoint create` proceed past the 10s default. However, `docker checkpoint create` still fails via a different codepath (see "What Didn't Work").

---

## What Didn't Work

### 1. `docker checkpoint create` with GPU containers

- Docker's checkpoint create calls CRIU via `swrk` (service worker) mode
- The CRIU CUDA plugin hangs when called through this path (times out at 300s)
- Same container checkpoints fine via `runc checkpoint` directly
- Root cause unclear — likely a PID namespace or ptrace interaction between Docker's checkpoint flow and the CUDA plugin's fork+exec of `cuda-checkpoint`
- **Workaround**: Use `runc checkpoint` directly, then copy files into Docker's checkpoint dir

### 2. `docker start --checkpoint` performance (261s)

- Even with Docker data-root on NVMe, restore takes ~261s
- Bare CRIU restore of same data takes 47s
- Docker adds ~220s of overhead recreating overlay filesystem, cgroups, network namespace, device mounts
- Page cache doesn't help — the overhead is container lifecycle, not I/O
- **This is why Modal built a custom runtime instead of using Docker/runc**

### 3. Docker checkpoint restore + network namespace

- When container uses bridge networking, `docker start --checkpoint` fails with `bind-mount /proc/0/ns/net: no such file or directory`
- Netns is cleaned up when container stops
- **Workaround**: Use `--network host`

### 4. vLLM multi-process + cuda-checkpoint

- vLLM v0.22 spawns EngineCore as a separate process (via multiprocessing)
- `cuda-checkpoint` can lock the parent but fails on the child: `"unknown error"`
- Known limitation: cuda-checkpoint supports single Linux process only, not process trees with shared GPU contexts
- **Workaround**: Use transformers directly (single-process) for GPU snapshot demo. vLLM for baseline only.

### 5. CPU-only snapshot saving import time

- CPU snapshot (CRIU after `import torch` without GPU) saves only ~6s of import time
- Post-restore still needs full model load + torch.compile + CUDA graph capture = 120s
- Net savings: negligible for the demo. Only useful when combined with GPU snapshot.

### 6. SOCI lazy loading for local images

- When image is already cached on the node, lazy loading adds zero value
- Only matters for first pull from registry to fresh nodes
- Not yet set up (SOCI snapshotter installed but not configured end-to-end)

---

## Key Measurements (Qwen2.5-7B-Instruct on L40S)

| Scenario | Time | Notes |
|----------|------|-------|
| Docker cold start (transformers, cached weights) | **131s** | imports + model load + warmup |
| Docker cold start (vLLM, cached weights) | **127s** | imports + model + compile + CUDA graphs |
| Docker cold start (vLLM, no weight cache) | **262s** | includes HF download |
| Bare CRIU restore (NVMe, cold) | **47s** | 17GB from disk |
| Bare CRIU restore (page cache warm) | **11.5s** | pages pre-loaded in RAM |
| Docker start --checkpoint (NVMe) | **261s** | Docker overhead dominates |
| Process-wake to first inference | **1.5s** | consistent across all restore methods |
| runc checkpoint (1.5B model) | **10s** | 5.2GB snapshot |
| runc checkpoint (7B model) | **82s** | 17GB snapshot |

---

## Architecture (How Modal Does It)

Modal doesn't use Docker. Their stack:
1. **gVisor (`runsc`)** as container runtime — not runc, not Docker
2. **gVisor's native checkpoint/restore** (`runsc checkpoint` / `runsc restore`) — not CRIU
3. **NVIDIA's `cuCheckpointProcess*` driver APIs** for GPU state
4. **Custom FUSE filesystem (ImageFS)** for lazy image loading
5. **Content-addressed cache** on local NVMe with aggressive read-ahead
6. **Background page-fault restore** — process starts before all pages loaded

This avoids Docker's 220s lifecycle overhead entirely. gVisor owns the kernel state so checkpoint/restore is clean and fast.

---

## Current State & Next Steps

**What we have proven**:
- GPU checkpoint/restore works end-to-end (CRIU 4.0 + CUDA plugin)
- Container checkpoint via runc works
- 1.5s from process-wake to inference (the "true serverless" number)

**What's blocking the ideal demo**:
- Docker's container lifecycle adds 220s overhead on restore (making it slower than cold start)
- Need either: (a) custom runtime that skips Docker's overhead, or (b) accept bare-process demo for video

**Next steps**:
1. Try `containerd` + `ctr` directly (bypass Docker daemon entirely)
2. Or use `runc restore` directly with a pre-built bundle (skip Docker's layer stack)
3. Set up SOCI snapshotter for lazy-pull-from-ECR demo
4. Record side-by-side video: baseline (131s) vs snapshot restore (11-47s)
5. Write LinkedIn Post 3 (GPU snapshots)

---

## Behavioral Guidelines

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them.
- If a simpler approach exists, say so.

### 2. Simplicity First
- No features beyond what was asked.
- No abstractions for single-use code.
- If you write 200 lines and it could be 50, rewrite it.

### 3. Surgical Changes
- Don't "improve" adjacent code.
- Match existing style.
- Every changed line should trace to the user's request.

### 4. Goal-Driven Execution
- Transform tasks into verifiable goals.
- Define success criteria before implementation.

---
> Source: [PotatoSpudowski/gpu-snapshot-experiment](https://github.com/PotatoSpudowski/gpu-snapshot-experiment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
