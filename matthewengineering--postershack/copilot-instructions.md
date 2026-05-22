## postershack

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

Postershack is a collection of AI image-generation apps. There are two independent apps, each with its own Docker setup and (in app2) Terraform for Azure deployment.

| App | Stack | Entry point |
|-----|-------|-------------|
| `app1_comfyui/` | ComfyUI backend + Streamlit frontend | `start.sh` / `docker compose up` |
| `app2/` | HuggingFace Diffusers + Gradio UI | `python diffuser.py` / `make run` |

---

## app1\_comfyui

**Run locally:**
```bash
cd app1_comfyui
cp .env.example .env   # fill in HF_TOKEN if needed
./start.sh             # or: docker compose up --build
```
- ComfyUI at `http://localhost:8188`
- Streamlit UI at `http://localhost:8501`

**Architecture:** Two containers orchestrated by `docker-compose.yml`. Streamlit talks to ComfyUI over the internal Docker network (`comfyui:8188`) via WebSocket for progress streaming and REST for image retrieval. Workflows are JSON files in `streamlit/workflows/` (currently `sd15.json`). `comfyui_client.py` injects prompts by finding nodes with `POSITIVE_PROMPT_PLACEHOLDER` / `NEGATIVE_PROMPT_PLACEHOLDER` text and the `KSampler` node for the seed — adding a new workflow means adding a JSON file with those placeholders and registering it in the `WORKFLOWS` dict in `app.py`.

**First run:** ComfyUI downloads the model on startup (~2–3 min). On Mac with Docker Desktop there is no GPU passthrough — generation runs on CPU (3–8 min/image).

---

## app2 (Diffusers + Gradio)

**Run locally (no Docker):**
```bash
cd app2
pip install -r requirements.txt
python diffuser.py
# UI at http://localhost:7860
```

**Docker via makefile:**
```bash
make build              # build image (diffuser-mvp:latest)
make run                # run container, UI at http://localhost:7860
make logs               # tail logs
make stop               # stop & remove container
make push IMAGE=myacr.azurecr.io/diffuser-mvp   # push to ACR
make shell              # bash into running container
```

**Terraform (Azure Container Apps):**
```bash
make init
make plan
make apply
```

**Architecture:** Single-file app (`diffuser.py`). Models are lazy-loaded on first Generate click and cached in `_pipeline_cache` (a module-level dict) so switching back to a previously used model is free. The `MODELS` dict at the top of the file is the single place to add/remove models — each entry has an `id` (HuggingFace repo ID) and an `azure` flag. When `azure=True` the pipeline is forced to `device="cpu"` regardless of available hardware, making it safe for Azure Container Apps with no GPU.

**Key dependency constraint:** `huggingface_hub` must be pinned to `==0.23.4`. Newer versions removed `cached_download`, which `diffusers==0.27.2` still imports.

**Azure deployment:** The Dockerfile sets `GRADIO_SERVER_NAME=0.0.0.0` (required for container networking). Use the **Fast — Azure / CPU-safe** model option for CPU-only Azure Container Apps; the other models need a GPU.

---
> Source: [MatthewEngineering/postershack](https://github.com/MatthewEngineering/postershack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
