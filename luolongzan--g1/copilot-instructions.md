## g1

> Grendel-GS is a research codebase for distributed 3D Gaussian Splatting. The

# Grendel-GS project guidance

## Project overview

Grendel-GS is a research codebase for distributed 3D Gaussian Splatting. The
main training path is Python/PyTorch with CUDA rasterization backends in Git
submodules. Read `README.md` before changing launch commands or training
semantics.

Key locations:

- `train.py` and `train_internal.py`: training entry points and main loop.
- `gaussian_renderer/`: rendering, workload partitioning, and distributed loss.
- `scene/`: cameras, datasets, and Gaussian model state.
- `arguments/`: command-line argument definitions.
- `submodules/`: CUDA rasterizers and other external dependencies.

## Working rules

- Use Chinese for explanations and progress updates unless the user requests
  another language. Keep code, commands, paths, and identifiers unchanged.
- Inspect `git status` and the relevant diff before editing. Preserve existing
  user changes and do not rewrite unrelated files.
- Treat submodules as separate repositories. Do not update submodule pointers,
  commit, push, or rewrite history unless explicitly requested.
- Do not download datasets, launch long training jobs, occupy GPUs, or kill
  processes unless explicitly requested. Check GPU availability immediately
  before any authorized GPU run.
- Never claim a performance or quality result without a completed run and its
  exact command, hardware/topology, dataset, logs, and comparison baseline.
  Keep single-GPU and multi-GPU evidence separate.
- Prefer focused verification first. For Python-only edits, run syntax checks
  on touched files. For CUDA or distributed changes, state clearly when full
  build or multi-GPU verification has not been run.
- Keep generated outputs, checkpoints, datasets, logs, and build artifacts out
  of version control.

## Common commands

The documented environment is `gaussian_splatting`:

```bash
conda activate gaussian_splatting
```

Single-GPU training:

```bash
python train.py -s <colmap-dataset> --eval
```

Four-GPU distributed training:

```bash
torchrun --standalone --nnodes=1 --nproc-per-node=4 train.py \
  --bsz 4 -s <colmap-dataset> --eval
```

Rendering and metrics:

```bash
python render.py -s <colmap-dataset> --model_path <model-directory>
python metrics.py --model_paths <model-directory>
```

Use the existing style in nearby code. Avoid broad formatting changes unless
the task explicitly requests them.

---
> Source: [LuoLongZan/g1](https://github.com/LuoLongZan/g1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
