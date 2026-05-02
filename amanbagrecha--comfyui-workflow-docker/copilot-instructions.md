## comfyui-workflow-docker

> - Always use `tmux` for long-running tasks such as model downloads, ComfyUI services, and multi-GPU runs.

# AGENTS.md

## General Conventions
- Always use `tmux` for long-running tasks such as model downloads, ComfyUI services, and multi-GPU runs.
- On rented or new Vast machines, prefer cloning `main`, running `bash scripts/bootstrap.sh`, and passing env vars for AWS profiles, data sync, and pipeline inputs instead of inventing a separate startup flow.

## Purpose
This repository runs a host-native 5-stage 360 panorama pipeline:
1. SAM3 tiled mask generation
2. ComfyUI inpainting via the local ComfyUI HTTP API
3. Laplacian sky replacement and panorama postprocess
4. Privacy blur
5. Per-stage counts and merged final outputs

Docker is no longer part of the runtime.

## Runtime Layout
- Python environment: `.venv/`
- ComfyUI checkout: `ComfyUI/`
- Shared model cache: `models/comfyui/` and `models/privacy_blur/`
- Per-GPU runtime data: `native_data/gpu<id>/`
- Logs: `logs/`

`setup_host_environment.sh` installs the pinned environment and links:
- `ComfyUI/models` -> `models/comfyui`
- `ComfyUI/custom_nodes/p2e` -> `p2e-local`

## Dependency Management
All Python dependencies are declared in `pyproject.toml` and pinned in `uv.lock`.
`setup_host_environment.sh` runs a single `uv sync` to install everything in one shot.

- Do **not** edit `requirements/` files — they no longer exist.
- To add or update a dependency: edit `pyproject.toml`, then run `uv lock` to regenerate `uv.lock`.
- `Comfy-Lock.yaml` is kept only for ComfyUI custom node git cloning (cm-cli). Pip entries there are ignored.
- `torch`, `torchvision`, and `torchaudio` are sourced from the `pytorch-cu128` index defined in `pyproject.toml` — never let anything reinstall these from PyPI.

## Main Entry Points
- `setup_host_environment.sh`
  - Clones ComfyUI, ComfyUI-Manager, and custom nodes; runs `uv sync` from `pyproject.toml`
- `scripts/bootstrap.sh`
  - Installs `aws` and `opencode` only if missing, optionally syncs data, and then runs `run_multi_gpu_pipeline.sh`
- `run_comfyui_cluster.sh`
  - Starts or reuses one ComfyUI tmux service per GPU
- `run_multi_gpu_pipeline.sh`
  - Splits `SRC` into shards and launches one shard worker per GPU
- `run_full_pipeline.sh`
  - Runs one shard end to end on one GPU

## S3 Parallel Downloader
For fast parallel downloads from S3, use `inpainting-workflow-master/s3_parallel_download.py`:
```bash
python inpainting-workflow-master/s3_parallel_download.py \
  --bucket BUCKET \
  --prefix "prefix1/" "prefix2/" \
  --dest DEST
```
Pass `--prefix` as space-separated values. Each prefix downloads into its own subfolder. `--prefix` is required.

## ComfyUI Services
`run_comfyui_cluster.sh` maps GPU ids to ports and per-GPU data roots:
- GPU 0 -> `http://127.0.0.1:8180` -> `native_data/gpu0`
- GPU 1 -> `http://127.0.0.1:8181` -> `native_data/gpu1`
- GPU N -> `http://127.0.0.1:(BASE_COMFY_PORT+slot)` -> `native_data/gpuN`

Each `run_comfyui_service.sh` process uses:
- `CUDA_VISIBLE_DEVICES=<gpu_id>`
- `--input-directory native_data/gpu<id>/input`
- `--output-directory native_data/gpu<id>/output`
- `--temp-directory native_data/gpu<id>`

## What `run_full_pipeline.sh` Does Per GPU
Each per-GPU shard job performs the following stages:

1. Stage shard input into `native_data/gpu<id>/input/<batch>`
2. Run `inpainting-workflow-master/$SAM3_SCRIPT` and write masks to `native_data/gpu<id>/output-sam3-mask/<batch>`
3. Run `inpainting-workflow-master/comfyui_run.py` against the local ComfyUI API and write outputs to `native_data/gpu<id>/output/<batch>`
4. Run `inpainting-workflow-master/postprocess.py` and write outputs to `native_data/gpu<id>/output-postprocessed/<batch>`
5. Run `inpainting-workflow-master/privacy_blur_infer.py` and write outputs to `native_data/gpu<id>/output-egoblur/<batch>`
6. Emit stage counts and JSONL events

## Useful Environment Variables
- `SRC`
- `FINAL_OUTPUT_DIR`
- `RUN_NAME`
- `SKIP_HOST_BOOTSTRAP`
- `GPU_IDS`
- `MAX_GPUS`
- `BASE_COMFY_PORT`
- `COMFYUI_HOME`
- `MODELS_ROOT`
- `MODELS_COMFYUI_DIR`
- `MODELS_PRIVACY_DIR`
- `NATIVE_DATA_ROOT`
- `FORCE_REPROCESS`
- `STOP_AFTER_STAGE`
- `SAM3_WORKERS`
- `POSTPROCESS_WORKERS`
- `PRIVACY_WORKERS`
- `STRICT_HARDLINK`
- `AWS_UPLOAD_PROFILE`
- `AWS_DOWNLOAD_PROFILE`
- `DOWNLOAD_S3_URI`
- `DOWNLOAD_DEST_DIR`

## Host-Side Paths
- Source dataset: user-provided `SRC`
- Staged shard input: `native_data/gpu<id>/input/<batch>`
- SAM3 mask output: `native_data/gpu<id>/output-sam3-mask/<batch>`
- Inpainting output: `native_data/gpu<id>/output/<batch>`
- Postprocess output: `native_data/gpu<id>/output-postprocessed/<batch>`
- Privacy blur output: `native_data/gpu<id>/output-egoblur/<batch>`
- Optional merged final output: `FINAL_OUTPUT_DIR/<run-name>/gpu<id>`

## Logging Artifacts
- Orchestrator logs: `logs/multigpu_<RUN_NAME>.log` and `logs/multigpu_<RUN_NAME>.events.jsonl`
- Per-shard logs: `logs/fullrun_<RUN_NAME>_g<gpu_id>.log` and `logs/fullrun_<RUN_NAME>_g<gpu_id>.events.jsonl`
- ComfyUI service logs: `logs/comfyui_g<gpu_id>.log`

## Checking Logs
```bash
# Live orchestrator status (upload progress, current sequence, failures)
tmux attach -t orch-next
# or tail the log directly:
grep -E "STATUS|OK|FAIL|Uploading|Tarring" /workspace/logs/orchestrator/orchestrator.log | tail -20

# Live GPU run progress
tmux attach -t run_<RUN_NAME>

# Check which tars uploaded vs. pending
grep -E "Upload verified|FAIL" /workspace/logs/orchestrator/orchestrator.log
```

## How To Run

### 1. Bootstrap (once per machine)
Installs AWS CLI, uv, configures AWS profiles, installs OpenCode.

```bash
AWS_DOWNLOAD_PROFILE=wasabi \
AWS_DOWNLOAD_ACCESS_KEY_ID=xxx \
AWS_DOWNLOAD_SECRET_ACCESS_KEY=xxx \
AWS_DOWNLOAD_REGION=us-east-1 \
AWS_DOWNLOAD_ENDPOINT_URL=https://s3.wasabisys.com \
AWS_UPLOAD_PROFILE=s3 \
AWS_UPLOAD_ACCESS_KEY_ID=xxx \
AWS_UPLOAD_SECRET_ACCESS_KEY=xxx \
AWS_UPLOAD_REGION=us-east-1 \
bash scripts/bootstrap.sh
```

### 2. Orchestrate (single entry point)
Downloads from Wasabi → runs pipeline → tars → uploads to S3 → deletes intermediates. Loops through all prefixes automatically.

```bash
bash orchestrate.sh <run_id_1> <run_id_2> ...
```

**All overridable env vars:**

| Var | Default | Description |
|---|---|---|
| `AWS_DOWNLOAD_PROFILE` | `wasabi` | Wasabi profile for input downloads |
| `AWS_UPLOAD_PROFILE` | `s3` | S3 profile for tar uploads |
| `WASABI_BUCKET` | `pano-bkp` | Source bucket on Wasabi |
| `S3_UPLOAD_PATH` | `s3://aipanoexport-batch2/panoramic_clean` | Upload destination |
| `S3_LOGS_PATH` | `s3://aipanoexport-batch2/logs/<hostname>` | Remote log destination — `<hostname>` is the machine's hostname, which on Vast.ai is a container ID (e.g. `08703c04a697`). Always override with a meaningful name. |
| `EVERY_NTH` | `3` | Download every Nth file |
| `DRY_RUN` | `0` | `1` = print plan only, no execution |

**Examples:**
```bash
# Test run against a different bucket, every 10th file
S3_UPLOAD_PATH=s3://my-test-bucket/test EVERY_NTH=10 DRY_RUN=1 \
bash orchestrate.sh 1021231_123123

# Production run, multiple sessions in tmux (always set S3_LOGS_PATH to identify the machine)
S3_LOGS_PATH=s3://aipanoexport-batch2/logs/vast-node-1 \
tmux new-session -d -s orch \
  'bash orchestrate.sh 1021231_123123 1034567_789012 1098765_432109'
tmux attach -t orch
```

### 3. Manual pipeline run (advanced)
```bash
RUN_NAME="my-run" \
SRC="/absolute/path/to/input_images" \
FINAL_OUTPUT_DIR="/absolute/path/to/final_outputs" \
./run_multi_gpu_pipeline.sh
```
Set `SKIP_HOST_BOOTSTRAP=1` if `setup_host_environment.sh` has already run.

---
> Source: [amanbagrecha/comfyui-workflow-docker](https://github.com/amanbagrecha/comfyui-workflow-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
