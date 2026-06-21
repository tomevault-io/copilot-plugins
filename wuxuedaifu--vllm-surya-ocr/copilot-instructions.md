## vllm-surya-ocr

> This repo serves a FastAPI OCR API from `api.py`. The API keeps the legacy

# Repository Instructions

## Project Context

This repo serves a FastAPI OCR API from `api.py`. The API keeps the legacy
request/response shape:

```json
{"data": ..., "message": "success", "code": 200}
```

Input validation is handled with Pydantic in `Info`. Keep external payload
fields stable unless the user explicitly asks for a breaking API change.

Primary endpoints:

- `POST /v1/api/ai/suya_ocr/`: legacy OCR route.
- `POST /v1/api/ai/suya_ocr_vllm/`: vLLM-backed OCR route.
- `GET /v1/api/ai/suya_ocr_vllm/health`: reports vLLM backend attachment.
- `POST /v1/api/ai/suya_text_det/`, `/suya_layout_det/`, `/suya_table_rec/`.

## Local Run

Use the conda Python on this host; Homebrew `python3` may not have the API
dependencies installed. For local direct `api.py` runs, start a vLLM OpenAI
server separately and point the API at it.

```bash
PYTHONDONTWRITEBYTECODE=1 \
PYTHONPATH=/path/to/suya-ocr-api \
SURYA_INFERENCE_BACKEND=vllm \
SURYA_INFERENCE_URL=http://127.0.0.1:8000/v1 \
SURYA_INFERENCE_AUTOSTART=false \
SURYA_INFERENCE_PARALLEL=8 \
SURYA_MAX_TOKENS_FULL_PAGE=6144 \
/opt/conda/bin/python api.py
```

The API listens on `http://127.0.0.1:5002`.

For Kubernetes/container deployment, use `Dockerfile`. It is based on
`vllm/vllm-openai:v0.20.1` and runs vLLM in the background, then `api.py` in
the foreground through `scripts/start_single_container.sh`.

## vLLM Backend Notes

The repo vendors the Surya source under `surya/`; runtime must not depend on
`/path/to/surya`. `vllm_tools.py` must set Surya/vLLM environment
defaults before importing `surya.settings` or Surya predictor classes.
Important defaults:

- `SURYA_INFERENCE_BACKEND=vllm`
- `SURYA_INFERENCE_URL=http://127.0.0.1:8000/v1`
- `SURYA_INFERENCE_AUTOSTART=false`
- `SURYA_INFERENCE_LOGPROBS=false`
- `SURYA_INFERENCE_MAX_RETRIES=1`
- `SURYA_MAX_TOKENS_FULL_PAGE=6144`
- `SURYA_MAX_BLOCKS_PER_PAGE=80`
- `SUYA_OCR_MODE=block`

The Dockerfile is tuned conservatively for a T4 machine:

- `VLLM_GPU_TYPE=t4`
- `VLLM_DTYPE=float16`
- `SURYA_INFERENCE_PARALLEL=8`
- `VLLM_MAX_NUM_SEQS=16`
- `VLLM_MAX_BATCHED_TOKENS=4096`
- `VLLM_GPU_MEMORY_UTILIZATION=0.85`

If running on A100 hardware, do not silently change the Dockerfile T4 defaults;
ask or document the deployment target first.

## Docker/NVIDIA

The production container must not run nested Docker. Kubernetes should provide
GPU access to the single container. The old Surya Docker auto-spawn path is
disabled unless `SUYA_ALLOW_NESTED_DOCKER=true` is explicitly set.

For host-level debugging only, Docker GPU runtime can be checked with:

```bash
docker info --format '{{json .Runtimes}}'
docker run --rm --runtime nvidia --gpus all --entrypoint nvidia-smi vllm/vllm-openai:v0.20.1 -L
```

If the named runtime is missing, use:

```bash
sudo nvidia-ctk runtime configure --runtime=docker --config=/etc/docker/daemon.json
sudo systemctl restart docker
```

`scripts/docker_nvidia_runtime_compat.sh` is legacy support for local debugging,
not the Kubernetes deployment path.

## Concurrency Benchmark

Run the comparison benchmark with the same image and settings when updating
results:

```bash
/opt/conda/bin/python concurrency_test.py \
  --image /path/to/surya/static/images/excerpt.png \
  --requests 16 \
  --concurrency 16 \
  --timeout 900 \
  --output concurrency_test_results.json
```

After a successful run, update:

- `concurrency_test_results.json`
- `IMPLEMENTATION_PLAN_AND_TEST_RESULTS.md`

Report latency columns as seconds. Mean latency is the average elapsed time per
request, not wall-clock time for the whole benchmark.

## Editing Guidance

- Use `rg`/`rg --files` for search.
- Use `apply_patch` for manual edits.
- Do not revert user changes or generated benchmark artifacts unless requested.
- Keep logging structured and include request IDs for API paths.
- Avoid import-time OCR work; model or backend attachment should happen only as
  needed by predictor setup and requests.

---
> Source: [wuxuedaifu/vllm-surya-ocr](https://github.com/wuxuedaifu/vllm-surya-ocr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
