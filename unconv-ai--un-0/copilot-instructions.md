## un-0

> See [README.md](README.md) for what this project is, the model recipe, and the

# Un0

See [README.md](README.md) for what this project is, the model recipe, and the
full setup / training / inference / evaluation commands. That is the source of
truth for both readers and contributors.

## Quick reference

```bash
uv sync --group dev    # core + tests + ruff
uv run pytest          # unit tests
uv run ruff check      # lint
```

## Hardware notes

- On Blackwell (sm_100+; B300 reports sm_103), the cuDNN 9.x bundled with
  `torch 2.11+cu128` has no valid SDPA execution plan, so the compiled DINO
  attention crashes with `No valid execution plans built`. Both training entry
  points call `common.disable_cudnn_sdp_on_blackwell()`, which falls back to
  flash, gated on compute capability `>= 10` so pre-Blackwell GPUs (H200, A100)
  keep cuDNN attention. After changing SDPA backends, clear the Inductor cache
  (`/tmp/torchinductor_*`) so a stale compiled graph doesn't keep calling the
  cuDNN attention op.

## Reproducing the released checkpoints: config truth

The released checkpoints (on the Hugging Face Hub) were trained with the configs
below. **These values are fixed** — reproduce against these tables, and note
where a value is a `build_*_model()` default or a module constant rather than a
CLI flag (so `--help` alone does not tell the whole story). CIFAR-10 and
ImageNet-64 are separate recipes and differ in several places (precision, weight
decay, queue size, parameterization, relativization, per-size LR).

### CIFAR-10 (`cifar10/n1024`, `n2048`, `n4096`)

`train_cifar10.py`'s **CLI defaults** reproduce `n2048`/`n4096`; the only knob to
change for `n1024` is `--lr`. Note the arch knobs (`num_steps`, `solver`,
`relativization`) are CLI args whose *defaults* differ from the
`build_cifar10_model()` builder defaults — the CLI default is the checkpoint
value.

| setting | n1024 | n2048 | n4096 | source |
| --- | --- | --- | --- | --- |
| `n_oscillators` | 1024 | 2048 | 4096 | `build_cifar10_model()` |
| params | 1.3M | 4.9M | 19.4M | (derived) |
| **`lr` (peak)** | **2.683e-3** | **1.389e-3** | **1.389e-3** | `--lr` |
| `batch_size` | 2048 | 2048 | 2048 | `--batch-size` |
| `epochs` | 1200 | 1200 | 1200 | `--epochs` |
| `precision` | fp32 | fp32 | fp32 | `--precision` (CLI default bf16; see note) |
| `weight_decay` | 1e-3 | 1e-3 | 1e-3 | `WEIGHT_DECAY` |
| `dino_weight` / `pixel_weight` | 1.0 / 0.004 | — | — | `--dino-weight` / `--pixel-weight` |
| `queue_size` / `num_pos` | 2048 / 64 | — | — | `--queue-size` / `--num-pos` |
| `queue_storage_dtype` | float32 | — | — | `--queue-storage-dtype` |
| `num_steps` | 10 | 10 | 10 | `--num-steps` (builder default 25) |
| `solver` | euler | — | — | `--solver` (builder default rk4) |
| `relativization` | mean_relative | — | — | `--relativization` (builder default ref_oscillator) |
| `parameterization` | standard | — | — | `--parameterization` |
| `n_conditional_oscillators` | 8 | 8 | 8 | `build_cifar10_model()` |
| `class_dropout_prob` | 0.1 | — | — | `build_cifar10_model()` |
| `betas` | (0.9, 0.95) | — | — | `BETA1/BETA2` |
| warmup / decay | 0.1 dur, linear→0 | — | — | `WARMUP_FRACTION` |
| `grad_clip` | 1.0 | — | — | `GRAD_CLIP_NORM` |
| `gamma` | 0.2 | — | — | `GAMMA` |
| seed | 42 | 42 | 42 | `--seed` |

### ImageNet-64 (`imagenet64/n6656`, `n10240`, `n16384`)

| setting | n6656 | n10240 | n16384 | source |
| --- | --- | --- | --- | --- |
| `n_oscillators` | 6656 | 10240 | 16384 | `build_imagenet64_model()` |
| params | 57M | 130M | 322M | (derived) |
| **`lr` (peak)** | **1.585e-3** | **1e-3** | **1e-3** | `--lr` |
| `batch_size` (per device) | 2048 | 2048 | 2048 | `--batch-size` |
| global batch (× 8 GPUs) | 16384 | 16384 | 16384 | — |
| `epochs` / `lr_schedule_epochs` | 3600 | 3600 | 3600 | `--epochs` / `--lr-schedule-epochs` |
| `precision` | bf16 | bf16 | bf16 | `--precision` |
| `weight_decay` | 0.0 | 0.0 | 0.0 | `WEIGHT_DECAY` |
| `dino_weight` / `pixel_weight` | 1.0 / 0.1 | — | — | `--dino-weight` / `--pixel-weight` |
| `queue_size` / `num_pos` | 128 / 64 | — | — | `--queue-size` / `--num-pos` |
| `queue_storage_dtype` | bfloat16 | — | — | `--queue-storage-dtype` |
| `num_steps` (euler) | 10 | 10 | 10 | `build_imagenet64_model()` |
| `n_conditional_oscillators` | 1 | 1 | 1 | `build_imagenet64_model()` |
| `parameterization` | mup | mup | mup | `build_imagenet64_model()` |
| `relativization` | ref_oscillator | — | — | `build_imagenet64_model()` |
| `class_dropout_prob` | 0.1 | — | — | `build_imagenet64_model()` |
| `betas` | (0.9, 0.95) | — | — | `BETA1/BETA2` |
| warmup / decay | 0.15 dur, linear→0 | — | — | `WARMUP_FRACTION` |
| `grad_clip` | 2.0 | — | — | `GRAD_CLIP_NORM` |
| `gamma` / `num_classes_per_step` | 0.2 / 64 | — | — | `GAMMA`, `NUM_CLASSES_PER_STEP` |
| drift temperatures | (0.02, 0.05, 0.2) | — | — | `DRIFT_TEMPERATURES` (`losses.py`) |
| seed | 42 | 42 | 42 | `--seed` |

In both tables `—` means identical to the leftmost (smallest) size. **The
smallest model used a higher LR** in each family (CIFAR `n1024` = 2.683e-3,
ImageNet `n6656` = 1.585e-3, from their own LR sweeps); the larger sizes share
one LR.

Two things these tables make explicit:

- **Many values are not CLI args** (or their CLI default differs from the model
  builder's). Architecture and training constants live in `build_*_model()`
  defaults, in the training scripts' module constants (`WARMUP_FRACTION`,
  `GRAD_CLIP_NORM`, `GAMMA`, `WEIGHT_DECAY`, …), and in `un0/losses.py`
  (`DRIFT_TEMPERATURES`, `EPS`, the DINO extractor internals). A
  reproduction-fidelity check must cover all of these, not just `--help`.
- **Precision** — the released CIFAR-10 checkpoints were trained in fp32, but the
  CLI default is bf16. bf16 trains fine and lands in the same FID ballpark, so
  there's **no need to switch to fp32** unless you want a bit-faithful
  reproduction. Do set the **per-size LR** explicitly for the size you reproduce.

### `--batch-size` is per device

`--batch-size` is the **per-device** batch; the global batch is
`batch_size × num_GPUs`. The reference global batch is 16384, which on 8 GPUs is
`--batch-size 2048`. A value that looks like a global batch will over-allocate
by `num_GPUs×`. Confirm against the README's example launch.

## ImageNet-64: adapting the data path (bring-your-own data)

`build_imagenet64_dataloader` in `un0/imagenet_data.py` is a **reference**
loader over a preprocessed PNG `ImageFolder` tree. To train on a different
backend (object store, WebDataset, a streaming loader, …), swap that one module
— everything downstream only needs the batch contract:

```python
{"data": Tensor,      # (B, 3*64*64) flat, float, range [-1, 1]
 "class_id": Tensor}  # (B,) int64, the true ImageNet class id (0..999)
```

Keep `class_id` as the **true class id**, not a remapped/sorted index — training
and the proxy FID's 1-to-1 label conditioning depend on it.

Gotchas when substituting a loader:

- **Iterable datasets and `set_epoch`.** The training loop calls
  `loader.sampler.set_epoch(epoch)` under DDP, which only exists on a map-style
  `DistributedSampler`. An `IterableDataset` (most streaming loaders) shuffles
  internally and its DataLoader sampler has no `set_epoch` — guard the call
  (`hasattr(loader.sampler, "set_epoch")`) or it crashes mid-epoch under DDP.
- **Per-rank sharding.** Map-style data uses a `DistributedSampler(rank,
  world_size)`. Many streaming loaders shard themselves from the environment —
  in that case do *not* also pass a sampler, or ranks will see overlapping data.
- **In-distributed dataset construction.** Some loaders run a collective
  (e.g. a barrier) at construction. Build them on **all ranks**, never inside an
  `if rank == 0:` block, or the other ranks hang waiting for a collective that
  never comes.

## ImageNet-64 evaluation: two different FID numbers

Do not conflate the two — they measure different things and are not comparable.

1. **In-training proxy FID** (logged by `train_imagenet.py`): clean-FID against
   custom validation statistics. It is a **convergence-tracking signal**, not the
   headline metric. It is computed on a subsampled generated set and is noisy
   run-to-run; its value drifts a few points between adjacent evals even when
   training is healthy. **Judge training health by `loss/total` and `grad_norm`,
   not by a single FID reading.** A single FID bounce with flat loss is sampling
   noise; only a *sustained* rise across several evals (or a loss/grad move) is a
   real regression.
2. **Headline ADM FID** (separate, offline): the field-standard ImageNet-64
   number from OpenAI's ADM evaluator against `VIRTUAL_imagenet64_labeled.npz`.
   This is what's comparable to ADM/EDM/DiT and what the README reports. It uses
   50k class-balanced samples; it is a separate TensorFlow tool (see
   [Evaluation](README.md#evaluation)) and is not run inside training.

Notes:

- **Class-balance the generated set** for any FID you report. The model is
  class-conditional, so the generated set's class marginal must match the real
  reference's, or FID measures class mismatch rather than image quality. The
  offline ADM path generates a class-balanced set; if you adapt the in-training
  proxy's conditioning labels, keep them balanced too.
- **clean-FID caches the real-image statistics** by name after the first eval
  (subsequent evals only generate + score). The cache key is the stats *name*,
  not a hash of the images — if you change the real reference set, use a new name
  or the stale stats are silently reused.
- **`torch.compile` coverage.** Dynamics and decoder are compiled inside
  `build_imagenet64_model()`; the DINO extractor and the drift-loss target are
  compiled in the training script. The first optimizer step pays a one-time
  compile cost (can be minutes); steady-state throughput is reached after. A
  fresh checkout / changed module path invalidates the inductor cache and
  triggers a full recompile — expected, not a hang.

## Practical notes for long runs

- **Smoke before committing to a multi-day run.** A 1-epoch run with
  `--fid-every-epochs 1` exercises the full path (data → queue warmup → loss →
  backward → distributed FID → checkpoint save) in minutes and catches
  environment/data issues before they cost days.
- **Checkpoints are resumable.** They carry model + optimizer + scheduler +
  epoch/step; `--resume <ckpt>` continues with the LR schedule intact. Set
  `--lr-schedule-epochs` to the *full* intended length when running a short
  prefix so per-epoch LR matches the full run.
- **Queue warmup.** With the per-class positive queue enabled, the first ~tens
  of steps report `queue_ready=0` and zero loss while the queue fills — expected;
  real loss begins once enough per-class samples are buffered.
- **Reference throughput** (sanity check, not a guarantee). ImageNet-64 `n16384`
  on **8× B200** runs at roughly **~70 s/epoch (~1.1 it/s, 78 steps/epoch at
  batch 2048/device)**, i.e. ~50 epochs/hr, so the full 3600-epoch run is on the
  order of **~3 days**. The first step is much slower (one-time `torch.compile`).
  Use this only to tell "healthy and on-pace" from "something is wrong" — it
  scales with GPU, model size, and batch.

---
> Source: [unconv-ai/Un-0](https://github.com/unconv-ai/Un-0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
