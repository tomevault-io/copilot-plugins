## verl

> `verl/` contains the Python package: trainers, workers, model engines, rollout backends, utilities, and configs. `examples/` holds runnable training scripts grouped by algorithm, including `examples/on_policy_distillation_trainer/`. `tests/` contains CPU, distributed, sanity, and end-to-end tests. `scripts/` contains conversion, checkpoint, and maintenance utilities. `docs/` contains user and contributor documentation. Generated artifacts such as `checkpoints/`, `logs/`, `outputs/`, and `wandb_offline/` should not be treated as source.

# Repository Guidelines

## Project Structure & Module Organization

`verl/` contains the Python package: trainers, workers, model engines, rollout backends, utilities, and configs. `examples/` holds runnable training scripts grouped by algorithm, including `examples/on_policy_distillation_trainer/`. `tests/` contains CPU, distributed, sanity, and end-to-end tests. `scripts/` contains conversion, checkpoint, and maintenance utilities. `docs/` contains user and contributor documentation. Generated artifacts such as `checkpoints/`, `logs/`, `outputs/`, and `wandb_offline/` should not be treated as source.

## Build, Test, and Development Commands

you can use this conda env for test or checking: 

Run focused tests while developing: source /mnt/shared-storage-user/p1-shared/leihaodi/miniconda3/bin/activate verl

```bash
pytest tests/test_protocol_on_cpu.py
pytest tests/trainer
```

Run lint checks with the project Ruff configuration:

```bash
ruff check verl tests scripts
```

Run example training scripts from the repository root, for example:

```bash
bash examples/on_policy_distillation_trainer/run_qwen_gsm8k.sh
```

## Coding Style & Naming Conventions

Use Python 3.10+ syntax and 4-space indentation. Keep line length near the configured Ruff limit of 120 characters. Prefer explicit names for distributed concepts such as `rank`, `world_size`, `tp_size`, and `micro_batch`. Test files should follow `test_*.py`; trainer scripts generally use `run_*.sh`.

## Testing Guidelines

Add or update tests near the changed subsystem. Use CPU tests for protocol, config, and utility changes; use `tests/special_distributed/` or `tests/special_e2e/` only when GPU/distributed behavior is required. For checkpoint or rollout changes, include at least a smoke command and document any hardware assumptions.

## Commit & Pull Request Guidelines

Recent history uses short imperative commit messages, for example `Fix SGLang speculative acceptance metric` or `Add DFLASH draft extraction script`. Keep commits scoped and avoid mixing generated artifacts with code. Pull requests should describe the motivation, list key changes, include test commands and results, and call out compatibility or migration risks. For AI-assisted contributions, the human submitter should review all changed lines and disclose assistance in the PR description.

## Security & Configuration Tips

Do not commit secrets, local dataset paths, W&B keys, or large checkpoints. Keep machine-specific settings in shell overrides or local scripts. When changing rollout or FSDP behavior, state expected GPU count and backend assumptions in the PR.

---
> Source: [bingyang-lei/verl](https://github.com/bingyang-lei/verl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
