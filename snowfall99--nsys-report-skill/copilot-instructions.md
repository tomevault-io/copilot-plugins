## nsys-report-skill

> Analyze an existing NVIDIA Nsight Systems (nsys) timeline report to find where time goes across an application — GPU idle/utilization, kernel & CUDA-API breakdown, memory transfers, stream concurrency, NVTX phases, and (multi-GPU) NCCL. Use when the user asks to read/analyze an nsys report (.nsys-rep / .qdrep / .sqlite), diagnose end-to-end GPU performance, find why training or inference is slow or the GPU is under-utilized, or check whether code is launch-bound or sync-bound — including Chinese variants ("分析 nsys 报告", "为什么 GPU 利用率低", "端到端为什么慢", "是不是 launch 受限").


# Skill: Nsight Systems Report Analysis (RTX PRO 6000 / Blackwell)

**When to use:** the user hands you an `nsys` report (or points at one) and wants to know where time goes — *is the GPU busy? what dominates? why is end-to-end slow?* Triggers: "analyze this nsys report", "why is GPU utilization low", "is this launch-bound", "where is the time going", "为什么这个 pipeline 慢", "帮我看一下 nsys".

**Target hardware (this repo):** NVIDIA RTX PRO 6000 Blackwell Workstation Edition — chip GB202, `sm_120` (CC 12.0), 188 SMs, ~1792 GB/s GDDR7, 96 GB, 134 MB L2. Timeline analysis is almost entirely hardware-agnostic — only peak-bandwidth interpretation and GPU-metrics-set availability are device-specific, and those are marked. The workflow works on any GPU Nsight Systems supports. (`TARGET_INFO_GPU` in the SQLite export carries the exact device for whatever report you're handed — read it, don't assume.)

---

## Scope — read this first

This skill **analyzes a report that already exists**. It does **not** build harnesses, write kernels, or run the application. Your inputs are one of:

- `report.nsys-rep` — current format (open in the GUI, or process with the CLI)
- `report.qdrep` — legacy format (convert/export the same way)
- `report.sqlite` — already exported

If — and only if — a report is missing, malformed, or missing the data you need (e.g. no NVTX, no GPU metrics), tell the user the exact `nsys profile` command to (re)collect it (see [Appendix: collection recipes](#appendix-collection-recipes)). Then stop and wait for the report. **Do not** profile on the user's behalf.

---

## Golden rule

**Read the report → diagnose → propose, in that order. Let the timeline tell you where time goes. Never guess.**

System-level performance almost always has a single dominant cause that the report shows in seconds: the GPU is idle waiting on the CPU, or thousands of tiny kernels are launch-bound, or a blocking `cudaStreamSynchronize` serializes everything, or a pageable `cudaMemcpy` stalls the pipeline. Don't hypothesize before you have the numbers, and don't write a wall of generic suggestions — rank them by **measured impact** (how much wall-clock each would actually recover).

---

## Workflow

### Phase 0 — Set up a run directory
Create `profile/<analysis_name>/` (one analysis = one directory; don't mix runs). Put the report in it and write all derived files (`stats/`, the `.sqlite`, `REPORT.md`) alongside. Resolve the report path and `export REPORT=...` for the commands below.

### Phase 1 — Frame the question
Before touching the data, write down: what is this report of (training step? inference request? a kernel pipeline?), how long is the captured window, and what does the user actually want answered ("why is it slow" vs "is the GPU saturated" vs "where's the memcpy time"). The answer determines which dimension matters.

### Phase 2 — Run the expert-system rules **first**
```bash
nsys analyze "$REPORT"
```
Nsight Systems ships a rules engine that auto-flags the usual suspects — GPU starvation, gaps in GPU utilization, frequent synchronization, pageable-memory copies, tiny kernels, etc. **Read these before doing anything by hand**; they often point straight at the answer. Treat each as a lead to confirm with numbers, not as the conclusion.

### Phase 3 — Pull the canned summaries
```bash
nsys stats --report cuda_gpu_kern_sum,cuda_api_sum,cuda_gpu_mem_time_sum,nvtx_sum "$REPORT"
```
This gives you the top kernels (total time / count / avg duration), the CUDA API breakdown, memory-op time, and NVTX ranges. `nsys stats --help-reports` lists every available report. Save outputs under `profile/<analysis_name>/stats/`.

### Phase 4 — Export to SQLite for anything deeper
The canned reports can't tell you GPU **idle %**, timeline **gaps**, per-stream **concurrency**, or any custom join. Export and query:
```bash
nsys export --type sqlite -o "${REPORT%.nsys-rep}.sqlite" "$REPORT"
```
See [SQLite quick reference](#sqlite-quick-reference) for the schema and copy-pasteable queries.

### Phase 5 — Work through the six analysis dimensions
See [Analysis dimensions](#analysis-dimensions). All matter, but on any given report only 1–2 dominate. Compute the headline number for each (even roughly) so you can rank.

### Phase 6 — Diagnose and write the report
Match the observed pattern to the [Diagnosis playbook](#diagnosis-playbook), then write `profile/<analysis_name>/REPORT.md`: the setup, the headline metrics, per-dimension findings **with cited numbers**, and optimization directions **ranked by expected wall-clock recovery** with caveats. Specificity is the deliverable.

---

## Analysis dimensions

| # | Dimension | Headline question | Where to look |
|---|---|---|---|
| 1 | **GPU utilization / idle** | What fraction of the captured window is the GPU actually doing work? Where are the biggest gaps? | SQLite: union of kernel+memcpy intervals vs wall; `gpu_gaps` query |
| 2 | **Kernel profile** | Which kernels dominate total GPU time? Are there many tiny ones (avg < ~5 µs)? | `cuda_gpu_kern_sum`; instance count × avg duration |
| 3 | **CUDA API / launch overhead** | How much host time is in `cudaLaunchKernel`, and how much is *waiting* (`cudaStreamSynchronize`, `cudaDeviceSynchronize`, `cudaEventSynchronize`)? | `cuda_api_sum` — separate *work* APIs from *wait* APIs |
| 4 | **Memory transfers** | H2D / D2H / D2D volume and time; pinned vs pageable; overlapped with compute or blocking? | `cuda_gpu_mem_time_sum` + `cuda_gpu_mem_size_sum`; memcpy `copyKind` |
| 5 | **Concurrency / overlap** | One stream or many? Do copies overlap compute? Does the GPU run while the CPU works? | SQLite: distinct `streamId`, overlapping intervals across streams |
| 6 | **Phases & communication** | Per-NVTX-range (iteration, fwd/bwd) timing; for multi-GPU, NCCL collective time and whether it overlaps compute | `nvtx_sum`, `nvtx_gpu_proj_sum`; `nccl_gpu_proj_sum` |

**Key distinction (don't skip):** in the API summary, a large `cudaStreamSynchronize` / `cudaDeviceSynchronize` is the CPU **waiting for the GPU** — it is a *symptom*, not the work. The real cost is whatever the GPU was (or wasn't) doing during that wait. Always cross-check API time against GPU-busy time.

---

## Diagnosis playbook

| Observed pattern | Likely cause | Direction |
|---|---|---|
| GPU idle high (busy < ~70% of wall); large gaps between kernels | CPU-bound / data-starved / serialized launches | Overlap host work, prefetch input, use CUDA Graphs, batch larger |
| `cudaLaunchKernel` dominates API time; thousands of kernels, avg duration tiny | **Launch-bound** | Fuse kernels, capture into a CUDA Graph, use persistent kernels |
| `cudaStreamSynchronize` / `cudaDeviceSynchronize` / `cudaEventSynchronize` dominate | Over-synchronization / false dependencies | Remove redundant syncs, sync on events not the device, overlap streams |
| Large memcpy time; copies are pageable and/or blocking (`cudaMemcpy` not `…Async`) | PCIe-bound, not overlapped | Pin host memory, use `cudaMemcpyAsync` on a copy stream, keep data resident on device |
| `cudaMalloc` / `cudaFree` appear in the steady-state loop | Allocator churn | Use a caching allocator / pre-allocate / memory pool |
| Single `streamId`; no copy↔compute or GPU↔CPU overlap | No concurrency exploited | Split independent work across streams; async copies |
| NCCL collectives are a large fraction and the GPU is idle during them | Communication-bound, no compute/comm overlap | Overlap comm with compute, fuse small collectives, check topology |
| Unified-memory page faults high (`um_*` reports) | UM thrashing | `cudaMemPrefetchAsync`, `cudaMemAdvise`, avoid oversubscription |

Each row is a hypothesis to confirm with the report's numbers, then size: *how much wall-clock would removing it recover?* Lead with the biggest.

---

## `nsys` command cheatsheet

```bash
# Expert-system rules (run these first)
nsys analyze "$REPORT"

# List every predefined stats report
nsys stats --help-reports

# Common summaries (comma-separated; --format table,csv and -o <dir> to save)
nsys stats --report cuda_gpu_kern_sum    "$REPORT"   # kernels: time / count / avg
nsys stats --report cuda_api_sum         "$REPORT"   # CUDA API time breakdown
nsys stats --report cuda_gpu_mem_time_sum "$REPORT"  # memory ops by time
nsys stats --report cuda_gpu_mem_size_sum "$REPORT"  # memory ops by size
nsys stats --report cuda_gpu_trace       "$REPORT"   # every GPU op, in order
nsys stats --report nvtx_sum             "$REPORT"   # NVTX range summary
nsys stats --report nvtx_gpu_proj_sum    "$REPORT"   # NVTX projected onto the GPU

# Export for custom analysis
nsys export --type sqlite -o report.sqlite "$REPORT"
```

> **Report-name drift:** report identifiers have changed across Nsight Systems versions (older names like `gpukernsum`, `cudaapisum` still work via aliases on some versions). If a `--report` name errors, run `nsys stats --help-reports` and use what that version lists.

---

## SQLite quick reference

All timestamps are in **nanoseconds**. String columns are interned — join `<col>Id` to `StringIds(id, value)`.

**Key tables**

| Table | Holds |
|---|---|
| `StringIds` | `id → value`; join everywhere names appear |
| `CUPTI_ACTIVITY_KIND_KERNEL` | GPU kernels: `start, end, streamId, deviceId, gridX/Y/Z, blockX/Y/Z, registersPerThread, staticSharedMemory, dynamicSharedMemory, graphId, shortName, demangledName, mangledName` |
| `CUPTI_ACTIVITY_KIND_MEMCPY` | GPU copies: `start, end, streamId, bytes, copyKind, srcKind, dstKind` |
| `CUPTI_ACTIVITY_KIND_MEMSET` | GPU memsets: `start, end, streamId, bytes, value` |
| `CUPTI_ACTIVITY_KIND_RUNTIME` | Host-side CUDA API calls: `start, end, globalTid, correlationId, nameId`. **In recent nsys (≥2024) driver calls (`cuLaunchKernel`, …) fold in here too — there is usually no separate `CUPTI_ACTIVITY_KIND_DRIVER` table, so this one query covers the whole API.** |
| `CUPTI_ACTIVITY_KIND_SYNCHRONIZATION` | explicit sync events: `start, end, streamId, syncType` (join `ENUM_CUPTI_SYNC_TYPE`) |
| `NVTX_EVENTS` | NVTX ranges/marks: `start, end, text, textId, eventType, globalTid` (name is `COALESCE(text, StringIds.value via textId)`) |
| `GPU_METRICS` | GPU-metrics samples — **only present if captured with `--gpu-metrics-device`** |
| `ENUM_CUDA_MEMCPY_OPER`, `ENUM_CUDA_MEM_KIND` | decode `copyKind` / `*Kind` ints — has `(id, name, label)`; `label` is the human string ("Host-to-Device") |
| `TARGET_INFO_GPU` | the device: `name, chipName, smCount, computeMajor/Minor, memoryBandwidth, totalMemory, l2CacheSize` |

> **Kernel names:** group by `shortName` (stable, e.g. `flash_fwd_kernel`). `demangledName` is the full per-template-instantiation signature and explodes into thousands of unique groups; keep it only for drilling into one kernel.

**Top kernels by total GPU time**
```sql
SELECT s.value AS kernel,
       COUNT(*)            AS instances,
       SUM(k.end - k.start) AS total_ns,
       AVG(k.end - k.start) AS avg_ns
FROM CUPTI_ACTIVITY_KIND_KERNEL k
JOIN StringIds s ON s.id = k.shortName          -- shortName, not demangledName (see note above)
GROUP BY s.value
ORDER BY total_ns DESC
LIMIT 15;
```

**Largest GPU idle gaps** (between consecutive GPU ops; treats kernels+copies as one timeline)
```sql
WITH ops AS (
  SELECT start, end FROM CUPTI_ACTIVITY_KIND_KERNEL
  UNION ALL
  SELECT start, end FROM CUPTI_ACTIVITY_KIND_MEMCPY
),
ordered AS (
  SELECT start, end, LAG(end) OVER (ORDER BY start) AS prev_end FROM ops
)
SELECT (start - prev_end) AS gap_ns, prev_end AS gap_start_ns
FROM ordered
WHERE prev_end IS NOT NULL AND start > prev_end
ORDER BY gap_ns DESC
LIMIT 20;
```
> Caveat: `LAG` over start handles a single serial timeline well; with heavy multi-stream overlap, compute a true interval union (`helpers/gpu_gaps.py` does this). For GPU-busy %, sum the *merged* op intervals and divide by `max(end) − min(start)`.

**CUDA API time, work vs wait**
```sql
SELECT s.value AS api,
       COUNT(*)            AS calls,
       SUM(r.end - r.start) AS total_ns
FROM CUPTI_ACTIVITY_KIND_RUNTIME r
JOIN StringIds s ON s.id = r.nameId
GROUP BY s.value
ORDER BY total_ns DESC
LIMIT 20;
```
Sort the result into *work* (`cudaLaunchKernel`, `cudaMemcpyAsync`) vs *wait* (`cudaEventSynchronize`, `cudaStreamSynchronize`, `cudaDeviceSynchronize`); the wait rows are the GPU-idle symptom, not the cost. A dominant `cudaEventSynchronize` is the classic "CPU blocked on the GPU" signature.

> **`helpers/summarize.py` runs all of the above** (GPU-busy %, top kernels, work-vs-wait API split, memcpy-by-kind, top NVTX) in one shot, and **`helpers/gpu_gaps.py`** finds the largest idle gaps and names the host call + NVTX range bracketing each — prefer them over re-typing these queries.

**CUDA Graphs caveat:** if `CUPTI_ACTIVITY_KIND_GRAPH_TRACE` is non-empty, kernels are replayed via `cudaGraphLaunch`, **not** individual `cudaLaunchKernel` calls. Graph-launched kernels still appear in `CUPTI_ACTIVITY_KIND_KERNEL`, but per-kernel launch-overhead reasoning no longer applies — the app already fused launches into graphs. Don't recommend "use CUDA Graphs" when the report shows they're already in use.

---

## Critical lessons (don't skip)

1. **`nsys analyze` does half the work — run it first.** The rules engine already names GPU starvation, sync overhead, tiny kernels, and pageable copies. Confirm each lead with numbers; don't re-derive it by hand.
2. **The canned summaries can't measure idle.** GPU-busy %, timeline gaps, and concurrency require the SQLite export. Reach for it as soon as the question is "where's the time going" rather than "which kernel is biggest".
3. **Distinguish *work* from *waiting*.** A huge `cudaStreamSynchronize` means the CPU is blocked on the GPU — the fix lives in what the GPU is doing (or not doing), not in the sync call itself.
4. **Many tiny kernels ⇒ launch-bound.** If avg kernel duration is a few µs and `cudaLaunchKernel` dominates host time, the GPU is starved by launch overhead — CUDA Graphs / fusion is the lever, regardless of how "fast" each kernel is.
5. **Timestamps are nanoseconds; report them as ms/µs and as % of the window.** "3.2 ms, 41% of the step" beats "3201184 ns". Always normalize against the captured wall-clock.
6. **Subtract profiler overhead and warmup.** The `PROFILER_OVERHEAD` table and the first iteration(s) distort totals — scope analysis to a steady-state NVTX range or a representative iteration when one exists.
7. **Don't delegate understanding.** Open the report yourself and cite specific values. Never write "the profile shows it's CPU-bound" — write "GPU-busy is 48% of the 80 ms window; the three largest gaps (4–6 ms each) sit inside `cudaStreamSynchronize` right after the dataloader NVTX range, so it's **input-starved**, not compute-bound." Fill in the real numbers. Specificity is the deliverable.

---

## File index

### Helpers (run these against a `.sqlite` export)

| File | Purpose |
|---|---|
| [`helpers/nsys_utils.py`](helpers/nsys_utils.py) | Shared library: connect, resolve `StringIds`, GPU info, ns formatting, interval-union |
| [`helpers/summarize.py`](helpers/summarize.py) | One-shot summary: GPU-busy %, top kernels, work-vs-wait API split, memcpy-by-kind, top NVTX, graph note |
| [`helpers/gpu_gaps.py`](helpers/gpu_gaps.py) | Largest GPU-idle gaps + the host call (`cuda*Synchronize`) and NVTX range bracketing each |
| [`helpers/parse_kernel.py`](helpers/parse_kernel.py) | **One kernel → per-shape latency suite.** Group a kernel's launches by launch shape (grid×block); emit `<base>_vllm_baseline.{md,jsonl}`. Decodes flash-attention grids into (batch, seqlen, heads). |

```bash
python3 helpers/summarize.py profile/<run>/report.sqlite --top 12
python3 helpers/gpu_gaps.py  profile/<run>/report.sqlite --top 15 --min-us 500
python3 helpers/parse_kernel.py profile/<run>/report.sqlite --kernel flash_fwd_kernel --out-dir profile/<run>/analysis
```

> `parse_kernel.py` reports **launch geometry** (grid×block), which nsys records —
> not tensor/problem dims (M,N,K, seqlen), which aren't in the trace. The
> flash-fwd decoder is the exception: it reconstructs (batch, seqlen, heads) from
> the grid using the FlashAttention launch formula + the `head_dim`/`kBlockM`
> parsed out of the kernel's demangled name.

### Reference docs (read when you need detail)

This file is a self-contained summary; each reference doc goes deeper.

| File | Purpose |
|---|---|
| [`reference/00-directory-layout.md`](reference/00-directory-layout.md) | Run-directory / naming conventions — one analysis = one directory |
| [`reference/01-workflow.md`](reference/01-workflow.md) | End-to-end checklist, report → diagnosis, with anti-patterns |
| [`reference/02-report-formats.md`](reference/02-report-formats.md) | `.nsys-rep` vs `.qdrep` vs `.sqlite`; `nsys stats` vs `export` vs `analyze` |
| [`reference/03-nsys-stats-reports.md`](reference/03-nsys-stats-reports.md) | Every predefined `nsys stats` report and when to use it |
| [`reference/04-sqlite-schema.md`](reference/04-sqlite-schema.md) | SQLite tables/columns + copy-pasteable query patterns |
| [`reference/05-analysis-dimensions.md`](reference/05-analysis-dimensions.md) | The six dimensions in depth, with calibrated example numbers |
| [`reference/06-diagnosis-playbook.md`](reference/06-diagnosis-playbook.md) | Pattern → cause → fix, with signals, thresholds, and sizing |
| [`reference/07-report-template.md`](reference/07-report-template.md) | Final `REPORT.md` structure |
| [`reference/08-rtx6000pro-notes.md`](reference/08-rtx6000pro-notes.md) | RTX PRO 6000 / Blackwell specifics (peak BW, PCIe, GPU-metrics) |
| [`reference/09-common-issues.md`](reference/09-common-issues.md) | Missing NVTX/metrics, warmup, profiler overhead, CUDA graphs, version drift |

---

## Appendix: collection recipes

Only when a report must be (re)collected. Hand these to the user — don't run them yourself.

```bash
# General CUDA app
nsys profile -t cuda,nvtx,osrt,cudnn,cublas -o report --stats=true ./app [args]

# Limit to a marked region (recommended for training/inference steady state):
#   wrap the region in cudaProfilerStart()/Stop() (or torch.cuda.profiler.start()/stop())
nsys profile -t cuda,nvtx,osrt -c cudaProfilerApi -o report ./app

# Multi-GPU / distributed: add nccl tracing
nsys profile -t cuda,nvtx,osrt,nccl -o report ./app

# Add lightweight GPU-metrics sampling (SM/mem util over time)
nsys profile -t cuda,nvtx --gpu-metrics-device=all -o report ./app
```
Tips: trace only what you need (`-t`), keep captures short (a few representative iterations), use a capture range to skip warmup, and prefer NVTX-annotated code so phases are legible in the timeline.

---
> Source: [Snowfall99/nsys-report-skill](https://github.com/Snowfall99/nsys-report-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
