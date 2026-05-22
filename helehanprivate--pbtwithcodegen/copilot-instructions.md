## pbtwithcodegen

> This repository is a LiveCodeBench-based code generation and repair project. The main code lives under `lcb_runner/`, with shell scripts under `script/` for running multi-step generation, repair, and evaluation workflows.

# Repository Guide

## Project Purpose

This repository is a LiveCodeBench-based code generation and repair project. The main code lives under `lcb_runner/`, with shell scripts under `script/` for running multi-step generation, repair, and evaluation workflows.

The `connect_to_qz/` directory is a separate control-plane helper for submitting jobs from a local machine to a relay/internal execution environment. It is not part of the benchmark runner itself, but it is important for running tasks on restricted internal machines.

## Top-Level Layout

- `README.md`: basic environment setup, dataset/model preparation notes, and quick-start commands.
- `requirements.txt`: Python dependencies for the benchmark runner.
- `lcb_runner/`: main Python package for benchmark loading, prompt construction, model execution, parsing, repair, and evaluation.
- `script/`: runnable shell entrypoints for baseline and quick-run experiments.
- `main_datasets/`: intended local dataset location. In the current checkout it may be empty or populated outside Git.
- `connect_to_qz/`: GitHub/shared-filesystem control-plane demo for remote/internal execution.
- `test_api.py`: small API/test helper at the repository root.
- `stress_gpu.py`: GPU stress/testing helper.

## Main `lcb_runner/` Structure

- `lcb_runner/benchmarks/`: dataset/scenario loaders and benchmark objects.
  - `code_generation.py`: code generation benchmark setup and dataset path handling.
  - `code_execution.py`: code execution benchmark support.
  - `test_output_prediction.py`: test output prediction benchmark support.
- `lcb_runner/prompts/`: prompt templates and prompt construction for different scenarios.
  - Includes prompts for code generation, self repair, checker/testcase generation, and test output prediction.
- `lcb_runner/runner/`: model runner implementations and CLI entrypoint.
  - `main.py`: main module used by `python -m lcb_runner.runner.main`.
  - `parser.py`: command-line argument definitions.
  - `base_runner.py`: common runner interface/behavior.
  - Provider-specific runners include OpenAI, Claude, Gemini, Cohere, Mistral, DeepSeek, Fireworks, vLLM, and local API variants.
  - `our_method.py`, `scenario_router.py`, and `runner_utils.py` contain project-specific orchestration logic.
- `lcb_runner/evaluation/`: execution, metric computation, pass@k helpers, score aggregation, and old-result checks.
- `lcb_runner/utils/`: shared utilities for extraction, multiprocessing, paths, and scenario definitions.

## Common Run Entry

`script/quick_run.sh` is the main quick-start experiment script:

```bash
bash script/quick_run.sh [GPU_NUMS] [MODEL_NAME] [DATASET_NAME]
```

Defaults:

- `GPU_NUMS`: `1`
- `MODEL_NAME`: `model/DeepSeek-R1-Distill-Qwen-32B`
- `DATASET_NAME`: `release_v5`

The script runs:

1. Initial `codegeneration` inference and evaluation.
2. A `checkerextend` repair/evaluation pass based on the code generation output.
3. Additional iterative `checkerextend` repair passes.

It sets `HF_DATASETS_OFFLINE=1` and `TRANSFORMERS_OFFLINE=1`, so local datasets and local model files are expected.

## `connect_to_qz/` Summary

`connect_to_qz/` implements a minimal three-side workflow:

```text
local machine -> GitHub code/control repos -> relay machine -> shared filesystem -> internal runner
```

The goal is to let a local coding agent submit auditable jobs without directly logging into or controlling the internal machine.

### Files

- `connect_to_qz/README.md`: architecture, setup instructions, operational commands, and phased plan.
- `connect_to_qz/qz_connect.py`: pure-Python CLI implementing init, submit, sync, runner, status, and logs commands.
- `connect_to_qz/qz_connect_config.json`: environment-specific configuration for code repo, control repo, shared root, internal code dir, model endpoint, wrapper, validation command, and timeouts.
- `connect_to_qz/scripts/claude_qwen.sh`: wrapper that configures Claude Code CLI environment variables for a Qwen-compatible Anthropic API endpoint.
- `connect_to_qz/.qz-work/`: local working area for cloned code/control repos and smoke-test artifacts. Treat it as generated state.

### Roles

- Local machine:
  - Develops code.
  - Pushes code branches/commits to the business code repository.
  - Submits job descriptions to the control repository.
  - Reads job status and log tails from the control repository.
- Relay machine:
  - Pulls GitHub code/control repositories.
  - Copies control-repo job inputs to the shared filesystem.
  - Copies internal runner outputs back to the control repository.
- Internal machine:
  - Reads jobs from the shared filesystem.
  - On first seeing a new shared job, writes `accepted.json` and waits one runner heartbeat before execution so code/control synchronization can settle.
  - Checks out the requested code commit.
  - Calls Claude Code through the configured wrapper/model endpoint.
  - Runs optional validation.
  - Writes full logs, status, summaries, and artifact indexes back to the shared filesystem.

### Current Config Highlights

The checked-in `qz_connect_config.json` currently points to:

- Code repository: `https://github.com/HeLeHanPrivate/PBTwithCodeGen.git`
- Control repository: `https://github.com/HeLeHanPrivate/connect_to_qz.git`
- Shared root: `/inspire/hdd/global_user/helehan-240108350133`
- Internal code directory: `/inspire/hdd/global_user/helehan-240108350133/projects/LiveCodeBench`
- Code branch: `main`
- Claude/Qwen endpoint: `http://127.0.0.1:8001`
- Model name: `qwen3.6-35b`
- Wrapper: `scripts/claude_qwen.sh`
- Validation command: `python -c "print('qz smoke command ok')"`

Because the config contains internal absolute paths, local execution may fail unless those paths exist or the config is adjusted.

### Important Commands

Initialize repositories and layouts:

```bash
cd connect_to_qz
python qz_connect.py --config qz_connect_config.json init
```

Submit a job from the local machine:

```bash
python qz_connect.py --config qz_connect_config.json submit --request "run internal validation"
```

Run relay sync once:

```bash
python qz_connect.py --config qz_connect_config.json sync-once
```

Run relay sync continuously:

```bash
python qz_connect.py --config qz_connect_config.json sync-loop --interval 30
```

Run internal jobs once:

```bash
python qz_connect.py --config qz_connect_config.json runner-once
```

Run internal jobs continuously:

```bash
python qz_connect.py --config qz_connect_config.json runner-loop --interval 30
```

Check status and logs:

```bash
python qz_connect.py --config qz_connect_config.json status <job_id>
python qz_connect.py --config qz_connect_config.json logs <job_id>
```

The old `relay-once` and `relay-loop` commands still exist as a legacy fixed-command smoke/demo path. The documented formal workflow is `sync-loop` on the relay machine plus `runner-loop` on the internal machine.

## Working Notes for Agents

- Prefer `rg` for repository search.
- Keep generated outputs, model outputs, datasets, and `.qz-work/` state out of source edits unless the task explicitly targets them.
- Be careful with `connect_to_qz/qz_connect_config.json`: it contains environment-specific internal paths and repository URLs.
- Do not assume local machine paths under `/inspire/...` exist.
- For benchmark changes, check `lcb_runner/runner/parser.py`, `lcb_runner/runner/main.py`, and the relevant scenario modules before changing shell scripts.
- For run-script changes, preserve offline-mode assumptions unless the user explicitly wants online HuggingFace/model access.
- GPU availability constraint: among the 4 GPUs, only GPUs `2` and `3` are available for this project. If they are temporarily occupied, wait 10 minutes and try again instead of switching to GPUs `0` or `1`.

---
> Source: [HeLeHanPrivate/PBTwithCodeGen](https://github.com/HeLeHanPrivate/PBTwithCodeGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
