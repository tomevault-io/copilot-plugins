## hyformer-pytorch

> This file describes the working conventions for this repository. Follow these rules when editing code, adding experiments, or extending the training pipeline.

# AGENTS.md

## Scope

This file describes the working conventions for this repository. Follow these rules when editing code, adding experiments, or extending the training pipeline.

## Repository layout

- `main_pytorch.py`
  Contains the core HyFormer implementation used as the reference backbone:
  - `SwiGLUFeedForward`
  - `CrossAttentionBlock`
  - `SequenceRepresentationEncoder`
  - `QueryBoostMixer`
  - `HyFormerLayer`
  - `HyFormerBackbone`
- `models/`
  Contains model wrappers built on top of the backbone. The current training model is `models/taac_hyformer.py`.
- `utils/`
  Contains reusable non-model logic:
  - common helpers
  - metric computation
  - TAAC dataset loading and tensorization
- `scripts/`
  Contains runnable entrypoints. The main training entrypoint is `scripts/run_taac2026_sample.py`.
- `outputs/`
  Runtime artifacts such as metadata JSON and checkpoints.
- `Test.md`
  User-facing command examples for the current training script.

## Architecture boundaries

- Keep backbone logic in `main_pytorch.py`.
- Keep downstream wrappers or task-specific heads in `models/`.
- Keep dataset parsing, feature conversion, metrics, and generic helpers in `utils/`.
- Keep `scripts/` thin. A script should mainly do:
  - argument parsing
  - loader construction
  - model/optimizer/scaler setup
  - training loop
  - checkpoint and metadata save

Do not move task-specific training logic back into `utils/` unless it is clearly reusable across scripts.

## Current model behavior

- The project uses a HyFormer-style backbone with:
  - non-sequential features mapped into `num_non_seq_tokens` global feature tokens
  - each behavioral sequence mapped into its own sequence-token stream
  - dedicated query tokens per sequence
  - stacked `Query Decoding -> Query Boosting` layers
- Query Generation:
  - initial queries are generated from concatenated non-sequential features and per-sequence pooled summaries
  - deeper layers reuse boosted queries from the previous layer
- Sequence encoder modes:
  - `longer`: compress sequence with short-query cross-attention
  - `full_transformer`: standard self-attention encoder
  - `swiglu`: attention-free feed-forward encoder
- Query Boosting:
  - concatenates decoded queries with non-sequence tokens
  - applies token mixing plus a per-token FFN
  - returns updated query and non-sequence tokens

When changing interaction semantics, keep `main_pytorch.py` and any model wrapper usage aligned.

## Training script contract

The current training script is `scripts/run_taac2026_sample.py`.

Expected supported behaviors:

- train from Hugging Face dataset or local parquet
- AMP support with CUDA autocast and optional GradScaler
- configurable sequence encoder via `--seq-encoder-type`
- configurable multi-sequence branch count via `--num-sequences`
- checkpoint save with timestamped filenames
- checkpoint resume via `--resume`
- save metadata to `run_metadata.json`

If new options are added, update:

- script argument parser
- `Test.md` usage examples
- this file if the workflow contract changes materially

## Checkpoint conventions

- Checkpoints must be saved with timestamped names, not fixed filenames.
- `--resume` should accept either:
  - a checkpoint filename under `output-dir`
  - an absolute checkpoint path
- Resume should restore, when present:
  - model weights
  - optimizer state
  - scaler state
  - last completed epoch
  - best validation metric tracked by the script
- Saving a checkpoint after resume must create a new file and must not overwrite the original checkpoint.

## Metrics and evaluation

- The current training target is classification trained with `CrossEntropyLoss`.
- The main validation selection metric is AUC.
- Keep metric code in `utils/metrics.py`.
- If metric definitions change, keep training logs and checkpoint metadata consistent with the new metric.

## Data handling

- TAAC dataset parsing logic lives in `utils/taac_data.py`.
- Support both:
  - the flattened dataset schema
  - the fallback/raw parquet-like schema when applicable
- Preserve schema autodetection unless there is a strong reason to remove it.
- Multi-sequence grouping should remain explicit in metadata so tensor shape is easy to trace.

## Coding expectations

- Prefer small, local changes over wide rewrites.
- Preserve current CLI behavior unless the change explicitly updates the interface.
- Keep shape semantics easy to trace.
- When changing tensor routing or token mixing logic, verify with at least:
  - `python -m py_compile ...`
  - a minimal forward or training run

## Validation checklist

For model or script changes, run the smallest validation that proves the path works.

Typical checks:

```bash
python -m py_compile main_pytorch.py models/taac_hyformer.py scripts/run_taac2026_sample.py
python main_pytorch.py
python scripts/run_taac2026_sample.py --max-rows 32 --epochs 1 --batch-size 8
```

If the change affects multi-sequence routing, query generation, or checkpointing, validate that specific path explicitly.

## Documentation upkeep

When behavior changes, keep documentation aligned:

- `README.md` for high-level repo description
- `Test.md` for command-line usage
- note files outside the repo only if explicitly requested

Do not silently let docs drift away from the implemented CLI or model behavior.

---
> Source: [WestbrookLong/Hyformer_Pytorch](https://github.com/WestbrookLong/Hyformer_Pytorch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
