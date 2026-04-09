## cosarad

> COSARAD (COntrastive Spatially-Aware Robotic Anomaly Detection) is a 3D point cloud anomaly detection system for manufacturing inspection. It uses contrastive learning to train a feature encoder, then detects anomalies by comparing test object patches against a memory bank of known-good features. The system is evaluated via 12-fold cross-validation split by object class, measuring O-ROCAUC (object-level) and P-ROCAUC (point-level).

# COSARAD — Claude Code Project Instructions

## What is this project

COSARAD (COntrastive Spatially-Aware Robotic Anomaly Detection) is a 3D point cloud anomaly detection system for manufacturing inspection. It uses contrastive learning to train a feature encoder, then detects anomalies by comparing test object patches against a memory bank of known-good features. The system is evaluated via 12-fold cross-validation split by object class, measuring O-ROCAUC (object-level) and P-ROCAUC (point-level).

Paper: ICRA 2026 (camera-ready submitted). Prior venue: ICPR.

## Repository structure

This is a Python ML research project with multiple backbone architectures. Discover which backbone/config is currently active by checking the pipeline runner constants (`scripts/run_train_eval_pipeline.py`) and the Hydra config referenced there.

### Entry points

The project has two main pipelines: **data preprocessing** (run infrequently, produces training data) and **ML training/evaluation** (run repeatedly during experimentation).

**Data preprocessing** (see `doc/supcon.md` and `doc/dataset.md`):
- `create_single_dataset.py` — Converts source datasets (Real3D-AD, MulSen, Anomaly-ShapeNet) into COSARAD format NPZ files.
- `register_pc_triplets.py` — Registers point clouds against reference templates. Produces transformation CSVs.
- `create_supcon_dataset.py` — Generates supervised contrastive patch tuples from registered point clouds. This is the final step that produces the training data consumed by the encoder.

**Registration** (see `registration/`):
- Currently uses ICP (`registration/icp_registration.py`) with multi-attempt wrapper (`registration/multiple_attempts_registration.py`). Used both in data preprocessing (batch registration) and at inference time (per-object registration against template). Registration accuracy directly impacts downstream anomaly detection quality. May be replaced with more robust methods (e.g. TEASER++, learning-based approaches).

**ML pipeline** (see `doc/training_validation_pipeline.md`, `doc/training.md`, `doc/evaluation.md`):
- `contrastive_learner_v3.py` — Training entry point. Hydra-configured, reads from `configs/`. Check the `TRAIN_CONFIG_NAME` constant in `scripts/run_train_eval_pipeline.py` to find the active config.
- `pipeline_reg_based.py` — Inference/evaluation entry point. Argparse-based. The `--encoder` flag selects which backbone to use.
- `scripts/compare_models.py` — Statistical comparison (Mann-Whitney U) between eval runs and baseline.

**Orchestration**:
- `scripts/run_train_eval_pipeline.py` — Pipeline runner. Orchestrates train→eval→compare stages via subprocess calls. Invoke with `python scripts/run_train_eval_pipeline.py <command>`.
- `scripts/` — Also contains standalone shell scripts for training and evaluation. See `doc/REPRODUCTION_GUIDE.md` for the authoritative list of current, clean scripts.

**Auxiliary**:
- `visualization/` — Scripts for debugging visualizations and paper figures. Not part of any pipeline.

### Key directories

```
configs/              Hydra YAML configs. Multiple configs exist for different backbones/losses.
                      Check scripts/run_train_eval_pipeline.py for which is currently active.
dataloaders/          PyTorch dataset/dataloader implementations
feature_extractors/   Encoder wrappers (one per backbone architecture)
registration/         Point cloud registration (currently ICP-based, may be upgraded)
external/             Third-party backbone code (e.g. riconv2, pct). Each may have
                      custom CUDA kernels that compile on first run.
docker/               Dockerfile, docker-compose.yml, entrypoint scripts
doc/                  Project documentation (pipeline, training, evaluation, dataset)
scripts/              Shell scripts for training, evaluation, analysis
visualization/        Point cloud and prediction visualization
utils/                Logging, debug, GradNorm, DDP utilities
weights/              Pre-trained model weights (organized by date)
```

### Files that should not be modified without understanding consequences

- `scripts/run_train_eval_pipeline.py` — Pipeline runner. Constants at the top control which Hydra config, model name, and dataset flags are used across all stages.
- `class_splits_*.py` — Cross-validation split definitions. The most recent one is active. Changing published splits breaks reproducibility.
- `constants.py` — Registration thresholds per object class. Tuned experimentally.
- `external/` — Forked backbones. Modifications may require recompilation of CUDA kernels.

## Development environment

### Docker (primary)

All training and inference runs inside Docker containers. Variants are defined in `docker/docker-compose.yml` — each targets a CUDA generation. Check `docker-compose.yml` for current variants and their GPU compatibility.

Container runs as non-root `appuser` with UID/GID remapping. Working directory: `/app` (repo mount). Data volume: `/data` (host: `/mnt/data/cosarad`).

```bash
# Typical invocation (substitute the service name for your GPU)
HOST_UID=$(id -u) HOST_GID=$(id -g) docker compose -f docker/docker-compose.yml run --rm <service> bash
```

### Local machine

- GPU: RTX 5090 (Blackwell)
- Data: `/mnt/data/cosarad` (symlinked as `./data`)
- Conda env name: check `*.yaml` environment files in repo root for current env name

### Lambda Labs (remote training)

- A100 80GB GPUs
- Persistent NFS at `/lambda/nfs/alexta-cosarad-<zone>`
- Ephemeral instances — setup via `source ./lambda/setup_lambda_env.sh`
- Code sync via rsync; data stored on Cloudflare R2

## Code conventions

### Python

- Python 3.11
- Type hints used inconsistently — prefer adding them when modifying functions
- Logging: use `logging` module, not print (except in training loop progress)
- Config: Hydra for training, argparse for inference
- Testing: `tests/` directory exists but coverage is minimal

### Naming patterns

- Training output: `best_<model_name>.pth` — the model name comes from the Hydra config's `model.name` field
- Training sentinel: `best_<model_name>.tmp` during training, renamed to `.pth` only on clean completion. The `.tmp` → `.pth` rename is the signal that training succeeded.
- Data splits: `train_classes_N` where N is 1–12, or `train_classes_all`
- Versioned scripts: files with version suffixes (e.g. `_v3`) indicate the current version. Lower versions are legacy — do not modify them.
- Config naming: `config_<backbone>_<loss>[_version].yaml`. Multiple configs coexist; only one is active in the pipeline at a time.

### Git

- `dvc.lock` must NOT be in `.gitignore` — DVC needs it for experiment tracking
- Large files (weights, data) tracked via DVC or stored on Cloudflare R2, not in git
- Commit messages: descriptive, no enforced format
- **Branch strategy**: `main` is the active development branch. Deprecated code (legacy learner versions, old scripts, superseded patch generators) will be preserved in a `v0.1` branch before removal from `main`. If you encounter files with old version suffixes still on `main`, they are candidates for this move — do not modify them.

## Documentation

The `doc/` directory contains comprehensive, maintained documentation for all project subsystems. Read the relevant doc file before making changes to a subsystem. Treat these as authoritative.

Key docs and what they cover:
- `doc/dataset.md` — COSARAD data format, directory layout, what NPZ files contain
- `doc/supcon.md` — Data preprocessing pipeline: point cloud registration, patch tuple generation, and training set construction. This is a prerequisite to training — the training set does not exist in raw form, it is synthesized from registered point clouds.
- `doc/training.md` — Contrastive encoder training
- `doc/evaluation.md` — Inference, evaluation, cross-validation structure
- `doc/training_validation_pipeline.md` — Pipeline runner (train→eval→compare automation)
- `doc/environment.md` — Docker and conda setup
- `doc/REPRODUCTION_GUIDE.md` — End-to-end reproduction instructions

## Known issues and gotchas

- **CUDA kernel compilation**: Backbones under `external/` depend on custom CUDA kernels (e.g. `pointops`). These compile on first run and are cached. If compilation fails, it's almost always an environment mismatch (wrong CUDA version, missing nvcc).
- **Hydra working directory**: Hydra changes cwd by default. The pipeline runner overrides `hydra.run.dir` to control where outputs land. If running manually without this override, outputs go to `./outputs/<date>/<time>/`.
- **filter_by_point_count**: Required for Real3D-AD (partial point clouds). Do NOT use for Anomaly-ShapeNet (full 3D shapes). Getting this wrong silently degrades accuracy.
- **YAML quoting for run IDs**: Run IDs containing underscores must be wrapped in single quotes in Hydra overrides, otherwise YAML parsers interpret them as integers and silently drop the underscore. The pipeline runner handles this automatically.

## Task-specific context

Load additional context files as needed for specific work areas:
@.claude/pipeline-runner.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alextarvo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alextarvo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
