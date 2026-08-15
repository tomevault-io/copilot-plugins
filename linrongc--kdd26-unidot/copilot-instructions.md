## kdd26-unidot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **A detailed architectural walkthrough already lives in [`.claude/CLAUDE.md`](.claude/CLAUDE.md)** (also auto-loaded). Read it for the full `UnifiedModel`/UniDot design — stage classes, sequence pipeline, FM Highway, DML sync rules. This file is the quick reference: commands + big-picture map.

## What this is

UniDot — a post-click conversion-rate (pCVR) model that unifies feature-interaction and sequential recommendation into one stackable block (`UnifiedModel` in `unified_model.py`). Streams Parquet, predicts a binary conversion label (`label_type == 2`). Self-contained training → checkpoint (with schema + config sidecars) → streaming inference.

## Setup

```bash
pip install torch pyarrow numpy scikit-learn tqdm tensorboard python-dotenv
# muon-optimize only needed for --dense_optimizer muon (the run.sh default)
cp .env.example .env   # then set TRAIN_DATA_PATH / TRAIN_CKPT_PATH / TRAIN_LOG_PATH / TRAIN_TF_EVENTS_PATH / SCHEMA_SRC_PARQUET
```

Requires Python 3.10+ and a CUDA GPU. Attention uses PyTorch SDPA — no FlashAttention install.

## Common commands

`config.MODE` (in `config.py`) selects the run profile: `local` (auto-pins one GPU + loads `.env`, the default), `tiny` (real data, single GPU, capped samples for A/B), `remote` (full multi-GPU DDP). `run.sh`/`run_dml.sh` read `MODE` and append the matching flags; they auto-select `torchrun` DDP when >1 GPU is visible, so pin a single device for smoke tests.

```bash
# Smoke test on a small parquet (single GPU)
CUDA_VISIBLE_DEVICES=0 bash run.sh --train_data_path /path/to/small.parquet \
    --split_by_time --num_epochs 2 --reinit_cardinality_threshold 0 --skip_num_slots 0

# Training — reference config, or override any flag
bash run.sh
bash run.sh --num_epochs 20 --dropout_rate 0.1

# Multi-path mutual learning (N mutually-distilling peers)
bash run_dml.sh

# Verify run.sh still parses after editing it
CUDA_VISIBLE_DEVICES=1 bash run.sh --help    # must exit 0

# Inference — rebuilds the model from the checkpoint's train_config.json
MODEL_OUTPUT_PATH=/path/to/checkpoint EVAL_DATA_PATH=/path/to/test.parquet \
    EVAL_RESULT_PATH=/path/to/predictions python3 infer.py
```

There is no separate lint/test suite; validation is empirical (AUC + LogLoss on a held-out split during training, then leaderboard). Inference-time knobs (batch size, `torch.compile`, autocast, matmul precision, DML serving mode) live in `config.py`, not on the CLI.

## Code map (big picture)

| File | Role |
|------|------|
| `unified_model.py` | The `UnifiedModel` (UniDot) architecture — five stage classes: `InputTokenizer`, `SequenceStage`, `UniDotBlock` (the stacked macro-layer), `FMHighway`, `ClassifierHead`. `remap_legacy_state_dict()` loads pre-refactor checkpoints. |
| `common.py` | Shared blocks — RoPE/attention, SwiGLU + `CondGatedSwiGLU`, LCB/NCB compression, `SequenceEmbedder`, encoder factory. |
| `dataset.py` | `PCVRParquetDataset` (streaming `IterableDataset`) + `FeatureSchema`, per-position weight fids, time bucketing. |
| `trainer.py` | `PCVRHyFormerRankingTrainer` — BCE training, dual optimizer (Adagrad sparse + AdamW/Muon dense), cold restart, EMA, self-contained checkpointing. |
| `dml.py` | `DMLTrainer`/`DMLModel` — Deep Mutual Learning. **Keep in sync with `trainer.py`** (see `.claude/CLAUDE.md` for the exact sync contract). |
| `train.py` | Entry point + argument parser. `build_model_from_args` is the single model builder (also used by `dml.py`). |
| `infer.py` | Streaming inference; `_FALLBACK_*_CFG` fills missing config keys for older checkpoints. |
| `muon.py` / `utils.py` / `config.py` | Muon optimizer / logging+early-stopping+seeding / launch mode + inference defaults. |
| `run.sh` / `run_dml.sh` | Reference training launchers (single-model / multi-path). |

## Repo-specific gotchas

- **`run.sh` is a live experiment** — token counts and flag values drift every few commits; never hardcode them. To disable a flag, **move its line into the commented block below `"$@"`** — never comment in place: a `#` inside the backslash-continuation chain silently truncates the command and drops later flags. Re-verify with `bash run.sh --help` (exit 0).
- **`dml.py` mirrors `trainer.py`.** Shared arg wiring lives in `train.py` helpers (`parse_seq_lens`, `build_pcvr_loaders`, `base_trainer_kwargs`) — add trainer/data flags there and both entry points pick them up. The only hand-implemented methods are `DMLTrainer._train_step` / `_evaluate_step` (must keep the 8-tuple return and DDP `no_sync` protocol); see `.claude/CLAUDE.md`.
- **"Run N epochs" means literally N epochs** — pass `--patience N` (or higher) so `EarlyStopping` (default `patience=5`) doesn't short-circuit an A/B comparison.
- **CRLF**: `.gitattributes` forces `*.sh`/`*.py` to LF on checkout. If bash on an online box hits `$'\r': command not found`, re-normalize with `git fetch && git reset --hard origin/main`.

---
> Source: [linrongc/kdd26_unidot](https://github.com/linrongc/kdd26_unidot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
