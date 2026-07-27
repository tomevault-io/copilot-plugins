## vortex-torch

> `vortex_torch` is a JIT-compiled sparse-attention framework that plugs

# vortex_torch — project primer for Claude Code

`vortex_torch` is a JIT-compiled sparse-attention framework that plugs
into sglang's decode loop. Users / AI agents write a **sparse-attention
submission** — two files in `submissions/` — and the framework compiles
them into Triton kernels at runtime.

## Objective

> **Strike the best tradeoff between accuracy (`mean@16`) and
> decoding throughput on AIME24.**

There is **no fixed quality floor** and no single number to maximise.
Both `mean@16` and `throughput` are objectives — the goal is to push
the **Pareto frontier** outward. A variant that buys throughput at
a small `mean@16` cost is useful; a variant that buys accuracy at a
small throughput cost is useful too. Map the frontier across a
batch by varying *along* the tradeoff: accuracy-leaning knobs
(looser `vortex_topk_val`, fewer `vortex_layers_skip`, bf16 KV) on
some variants, throughput-leaning knobs (tighter `topk`, more layer
skips, fp8 `kv_cache_dtype`, `approxTopK(tolerate_ratio=…)`,
`mem_fraction_static → 0.9`) on others. Pick winners by where
they sit on the `(throughput, mean@16)` plane against the running
best in `memory.md §5`, not by clearing a fixed bar.

## Inventing beyond the literature

The `papers/` folder and [papers/guide.md](papers/guide.md) cover
what's already published — sinks, heavy hitters, channel sparsity,
low-rank K, LSH sampling, dual-band centroids. Treat them as
**seeds, not a menu.** A winning flow does not need a citation.

**Novelty bar.** Algorithmic innovation is the primary objective.
Every batch must reserve **at least one slot** (aim for two) for a
*genuinely novel* variant — an idea from §16.2 (untried knobs),
§16.3 (inversions), §16.4 (first-principles), or the framework's own
op set. Paper replicas and §16.1 combinations are catalog-adjacent
and do not fill the novelty slot.

**The remaining 2–3 slots should use `papers/guide.md` §16.5
techniques** — catalog-adjacent parameter sweeps (`vortex_topk_val`,
`approxTopK`, layer-skip patterns, fp8/bf16 KV, etc.) that are
explicitly encouraged for non-novelty slots. They map the Pareto
curve around the novel variant and give the measured context needed
to judge whether the novel idea is actually buying something.

## Where the instructions live

All authoritative content lives under [AI/](AI/). Read in order:

1. [AI/AGENTS.md](AI/AGENTS.md) — the full submission contract, rules,
   budget / BOS / layer-skip semantics, benchmark protocol.
2. [AI/tutorials/overview.md](AI/tutorials/overview.md) — 5-minute map.
3. [AI/tutorials/program_create_cache.md](AI/tutorials/program_create_cache.md)
4. [AI/tutorials/program_forward_cache.md](AI/tutorials/program_forward_cache.md)
5. [AI/tutorials/program_forward_indexer.md](AI/tutorials/program_forward_indexer.md)
6. [AI/tutorials/cache_op.md](AI/tutorials/cache_op.md) — indexer-side
   op math reference.
7. [AI/tutorials/indexer_op.md](AI/tutorials/indexer_op.md) — cache-side
   op math reference.
8. [papers/guide.md](papers/guide.md) — synthesis of the ten
   sparse-attention papers in `papers/`. §14 = catalog of
   known-good submission ideas; **§16 = prompts for inventing
   flows that no paper here explores.**

Framework-internal deep dives live in
[AI/developer_guides/](AI/developer_guides/) — needed only if you are
modifying the compiler itself, not when writing a submission.

## Hard constraints

- **No native torch ops** inside `create_cache` / `forward_cache` /
  `forward_indexer`. Every tensor goes through
  `vortex_torch.indexer.*` / `vortex_torch.cache.*` ops. `.view`,
  `.sum(dim=...)`, elementwise torch, etc. will not compile.
- **Each op instance is one call site.** `self.mul_a = Multiply()`
  and `self.mul_b = Multiply()` — do not share.
- **Do not declare `"k"` or `"v"`** in `create_cache`; they are
  auto-provided.
- **`forward_indexer` must end in `topK(score, o, ctx=ctx)` or
  `approxTopK(tolerate_ratio=...)(score, o, ctx=ctx)`** — the
  score must be RAGGED `[S, 1, 1]`. `approxTopK` is a faster
  adaptive 8-bit radix variant; `tolerate_ratio ∈ [0.0, 1.0]`
  where `0.0` = exact, higher = cheaper but looser.
  *Trtllm-only alternative:* with
  `"vortex_attention_backend": "trtllm"` you can instead end in
  `Union()((bt_a, sl_a), (bt_b, sl_b), o, ctx=ctx)` fed by two
  `TopK(k=...)(score, ctx=ctx) → (block_tables, seqlens)` calls.
  `TopK` / `Union` assert at profile time if used under flashinfer
  (see `AI/tutorials/indexer_op.md §9b`).
- **Cache-side reductions support `dim ∈ {1, 2}` only.** Cross-block
  reductions (`dim=0`) belong on the indexer side.
- **If a field is read+written across steps via `Load`/`Save`, zero
  it with `CFill(0.0)` in `forward_cache`.**
- **If `forward_indexer` uses `Save(...)`, the engine JSON MUST set
  `"disable_radix_cache": true`** (default `false`). sglang's
  prefix-radix cache otherwise shares per-request Save'd state
  across requests with matching prompt prefixes, corrupting
  Save/Load values. `check_engine_config` rejects the violation.

## Environment — activate the `vortex_v1` conda env first

Every python invocation in this project (`check_engine_config`,
`run_submission_aime24.py`, the pre-flight loops in the slash
commands, etc.) expects the **`vortex_v1`** conda environment.
**Activate it once at session start** before running any of the
bash snippets below:

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate vortex_v1
python -c "import sys; print(sys.executable)"   # expect a path under .../envs/vortex_v1/
```

If `conda activate` isn't available in the current shell (e.g. a
non-interactive sub-shell that didn't source the conda profile),
fall back to `conda run -n vortex_v1 python ...` for every
python call. Either form is acceptable; what matters is that the
running interpreter is the one inside `vortex_v1`.

## Running the benchmark — policy

**Every batch is exactly 4 variants.** That fixed width is what
makes orthogonal-knob sweeps and Pareto-frontier mapping work.
Parallelism depends on how many GPUs are free *now* — the host
may share GPUs with other users:

- `N >= 4` free GPUs → run all 4 variants in parallel, one per GPU.
- `0 < N < 4` free GPUs → run the 4 variants in **waves of N**
  on the available GPUs (sequential fallback). With `N = 1` this
  is fully serial; `N = 2` runs `2 + 2`; `N = 3` runs `3 + 1`.
  The batch still produces 4 results; it just takes longer.
- `N == 0` (`free_gpus.sh` returns empty / exits 1) → **hard wait**,
  do not launch.

Detect free GPUs at the start of every batch:

```bash
FREE_GPUS=($(algorithm_scientist/free_gpus.sh)) || {
    echo "no free GPUs — wait, do not launch" >&2; exit 1
}
N=${#FREE_GPUS[@]}
BATCH_SIZE=4
PARALLEL=$N
[ "$PARALLEL" -gt "$BATCH_SIZE" ] && PARALLEL=$BATCH_SIZE
echo "free GPUs: ${FREE_GPUS[*]}  (N=$N, parallel=$PARALLEL, batch=$BATCH_SIZE)"
```

`free_gpus.sh` excludes GPUs that have a compute process running
on them or memory.used ≥ 1024 MiB (override via
`free_gpus.sh <mib>`). Empty result (exit 1) ⇒ hard wait. Any
`N >= 1` is launchable; `N < 4` just serialises into multiple
waves on the available GPUs.

**File layout.** All submissions you write live under
`submissions/<tag>/`, where `<tag>` is your agent identifier
(default: a sanitized lowercase form of your model name, e.g.
`claude_opus_4_7`). Within that dir, batched runs use the
convention `batch_<x>_id<y>.{py,json}` (`<x>` = batch index,
`<y>` = per-variant index in `0…3`). Single-variant runs are
debug-only. Each batch:

1. **4 orthogonal variants** —
   `submissions/<tag>/batch_<x>_id0.{py,json}` …
   `submissions/<tag>/batch_<x>_id3.{py,json}` — varying
   different knobs. The slot count is fixed at 4 regardless of
   how many GPUs are free; only the parallelism changes.
2. **Cheap local pre-flight first** for all 4 (CPU, no GPU):
   ```bash
   TAG=<your_agent_tag>; BATCH=<batch_index>
   for y in 0 1 2 3; do
     python -c "from vortex_torch.engine.sgl import check_engine_config; check_engine_config('submissions/${TAG}/batch_${BATCH}_id${y}.json')"
   done
   ```
   Refuse to launch any variant whose pre-flight fails.
3. **RULER pre-filter — quick quality gate (≥ 0.85).** Before
   spending 20–60 minutes on AIME24, run `algorithm_scientist/run_ruler.py`
   on each variant. Any variant scoring below **0.85 accuracy** on
   `examples/validation.jsonl` has structurally broken attention —
   fix it (widen `vortex_topk_val`/`vortex_topk_ratio` or revise the
   indexer scoring), re-pre-flight, and re-run RULER until all 4 pass.
   ```bash
   for y in 0 1 2 3; do
     CUDA_VISIBLE_DEVICES=${FREE_GPUS[0]} \
       python algorithm_scientist/run_ruler.py \
         --config "submissions/${TAG}/batch_${BATCH}_id${y}.json"
   done
   ```
   Results land in `summary_ruler_submissions/<tag>/<stem>/latest.json`.
4. **Launch the 4 variants in waves of `PARALLEL = min(N, 4)`**,
   each child pinned via `CUDA_VISIBLE_DEVICES`, with `wait`
   between waves so a wave's GPU is free before the next one
   reuses it:
   ```bash
   LOGDIR="logs/submission/${TAG}_batch_${BATCH}_$(date +%Y%m%d_%H%M%S)"
   mkdir -p "$LOGDIR"
   BATCH_SIZE=4
   PARALLEL=$N
   [ "$PARALLEL" -gt "$BATCH_SIZE" ] && PARALLEL=$BATCH_SIZE
   for start in $(seq 0 $PARALLEL $((BATCH_SIZE - 1))); do
       end=$((start + PARALLEL))
       [ "$end" -gt "$BATCH_SIZE" ] && end=$BATCH_SIZE
       for y in $(seq $start $((end - 1))); do
           cfg="submissions/${TAG}/batch_${BATCH}_id${y}.json"
           gpu="${FREE_GPUS[$((y - start))]}"
           stem=$(basename "$cfg" .json)
           CUDA_VISIBLE_DEVICES=$gpu \
               python algorithm_scientist/run_submission_aime24.py --config "$cfg" \
               > "$LOGDIR/gpu${gpu}_${stem}.out" \
               2> "$LOGDIR/gpu${gpu}_${stem}.err" &
       done
       wait
   done
   ```
   The id `<y>` is the variant slot (0…3), NOT a GPU index —
   the actual GPU is `FREE_GPUS[$((y - start))]` within each wave.
   When `N >= 4` there is exactly one wave of 4 (fully parallel,
   identical to the old behaviour). When `N < 4` the loop runs
   ⌈4/N⌉ waves; the wall-clock cost scales accordingly. Each
   child writes its result into
   `summary_submissions/<tag>/<stem>/<timestamp>__<hash>.json`
   and updates `latest.json` on its own. The runner mirrors the
   config's path under `submissions/` into `summary_submissions/`,
   so `submissions/<tag>/batch_<x>_id<y>.json` becomes
   `summary_submissions/<tag>/batch_<x>_id<y>/...` — per-agent
   isolation, no collisions between agents that happen to use
   the same `batch_x_idy` stem.

5. **One batch at a time on the free GPUs.** Do not launch a
   second batch while the first (any wave) is still running, and
   do not try to "fill the gaps" by launching extra variants on
   GPUs another user freed mid-batch — both contend for memory
   and either OOM or thrash. Use `jobs` (or `ls -lt
   summary_submissions/<tag>/*/latest.json`) to see how many
   children are still alive while you wait.

## While you wait (20–60 min per batch; kill any child > 60 min)

Idle is not an option. Each polling cycle, do one of:

- **Read.** Priority: `AI/tutorials/` → `AI/developer_guides/` →
  `papers/` → `vortex_torch/flow/algorithms.py` →
  `vortex_torch/{indexer,cache}/*` → `csrc/`. After each file,
  append one insight to `algorithm_scientist/memory.md` §7.
- **Invent.** Open `papers/guide.md` §16 and pick a §16.2
  (untried knob), §16.3 (inversion), or §16.4 (first-principles)
  prompt — *not* §16.1 combinations, those are catalog-adjacent
  and don't fill the novelty slot. Better still: come up with
  a hypothesis derived from the framework's op set itself that
  doesn't fit any §16 sub-bucket. Sketch a one-sentence
  hypothesis + cache/indexer ops, naming the specific op or
  behaviour exploited. Aim for two such sketches per wait cycle.
- **Design (don't launch) the rest of the next batch.**
  Pre-flight the 4 candidates so they're ready to fire the
  moment `wait` returns. Concurrent batches would OOM the
  shared GPUs.
- **Analyse children early.** As individual `latest.json` files
  appear (children finish at slightly different times), pull
  their `mean@16` / `throughput` and start filling a §2
  sub-section in memory.md. Close the §1 row when all 4 are
  in, then update §3 (hypotheses) / §4 (anti-patterns) / §5
  (winners).

## Persistent state — `algorithm_scientist/memory.md`

The conversation evaporates; `memory.md` does not. Read it at the
start of every session and write to it before stopping. Any batch
submission and any batch completion must mutate it.
   When the job finishes, each run is written into a
   per-submission subfolder so iterations never collide:

   ```
   summary_submissions/<name>/
       <timestamp>__<content_hash>.json   # full summary + embedded .py/.json
       latest.json                        # symlink → newest run
       INDEX.jsonl                        # one-row-per-run index
   ```

   The content hash is `sha256(config.json || module.py)` truncated
   to 12 chars — same code → same hash → you can see re-runs
   at a glance. Read `summary_submissions/<name>/latest.json`
   after a run, and on failure read the per-child log under
   `logs/submission/batch_<TS>/gpu<i>_<stem>.{out,err}` (or the
   `logs/submission/single_<TS>/<stem>.{out,err}` produced by
   `/benchmark`).

## Kickoff prompt for new sessions

To boot a fresh Claude Code session straight into the long-horizon
iterate loop, paste the prompt block from
[algorithm_scientist/iterate_kickoff.md](algorithm_scientist/iterate_kickoff.md)
into the new session's first message. The agent will identify
its tag, bootstrap from this primer + AGENTS.md + tutorials +
papers/guide.md + memory.md, and start the first batch
autonomously. State lives in `algorithm_scientist/memory.md` and
the `submissions/<tag>/` / `summary_submissions/<tag>/` trees,
so any later session resumes cleanly from the same prompt.

## Slash commands available in this session

- `/new-submission <name>` — scaffold a new submission pair.
- `/preflight <name>`      — run the cheap local pre-flight.
- `/batch-benchmark <n1> <n2> <n3> <n4>` — launch the 4-variant batch on the currently-free GPUs (parallel when `N >= 4`, otherwise waves of `N`; the only sanctioned benchmark command).
- `/review <name>`         — audit a submission against AGENTS.md rules.
- `/iterate <name>`        — kick off a full auto-iteration loop (4 variants per batch on the currently-free GPUs, one batch at a time, updates memory.md).
- `/innovate <N> [theme]`  — *innovation-draft mode*: produce N genuinely-novel submissions in one shot, all required to compile, no benchmark loop, no memory.md mutation. See AGENTS.md §5f.
- `/benchmark <name>`      — *debug only*: run a single variant directly. Do not use in normal workflow.

## Subagents available

- `vortex-submission-writer` — drafts and iterates on a submission.
- `vortex-submission-reviewer` — audits a submission pair for
  rule violations without editing anything.

Use `Task(subagent_type="vortex-submission-writer", ...)` from the main
agent or invoke via slash command.

---
> Source: [Infini-AI-Lab/vortex_torch](https://github.com/Infini-AI-Lab/vortex_torch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
