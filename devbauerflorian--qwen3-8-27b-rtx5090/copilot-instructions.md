## qwen3-8-27b-rtx5090

> Guidance for AI agents working in this repository. This repo is a

# AGENTS.md

Guidance for AI agents working in this repository. This repo is a
**deployment recipe**, not a framework: a single, fully pinned Docker stack
that serves **Qwen3.8-27B (NVFP4, ~18.8 GB)** on **one RTX 5090 (32 GB)** via
vLLM, with an OpenAI-compatible API (reasoning + auto tool choice, 256K
context), a Basic-auth gateway, and a Prometheus/Grafana/DCGM monitoring
sidecar set.

- Human docs: [`README.md`](README.md) (canonical, keep in sync), [`CONTRIBUTING.md`](CONTRIBUTING.md) (commit style).
- Everything that can break a rebuild is **pinned** (see *Pinning discipline* below). Never "modernize" a version casually.

## Commands

| Task | Command |
|---|---|
| Install host prerequisites + download weights (~19 GB, pinned HF revision) | `sudo ./setup.sh` |
| Configure | `cp .env.example .env` then edit (secret: `.env` is gitignored — never commit it) |
| Build + start the stack | `docker compose up -d --build` |
| Watch boot | `docker logs -f vllm` (ready when logs show `Application startup complete`) |
| Smoke test | `curl http://localhost:8020/v1/models` (add `-H "Authorization: Bearer $VLLM_API_KEY"` if the key is set) |
| Smoke, via gateway | `curl -u "$METRICS_USER:$METRICS_PASSWORD" http://localhost:8030/v1/models` |
| Context benchmark | `.venv_download/bin/python benchmarks/context_bench.py` (add `--cached` for prefix-cache deltas, `--quick` for sanity) |
| Throughput benchmark | `.venv_download/bin/python benchmarks/parallel_bench.py` |

There is no test framework or linter. **Verification = the stack comes up,
the smoke curl works, and the benchmark suites pass.** Run benchmarks only on
an idle GPU; results land in `benchmarks/results/` (gitignored, except
`.gitkeep`) and both suites print a paste-ready markdown table for the README.
Bench scripts need `httpx` (+ optional `tokenizers`), which live in
`.venv_download/` (created by `setup.sh`; `httpx` falls back to stdlib
`urllib` if missing).

## Repo map

| Path | What |
|---|---|
| `Dockerfile` | vLLM image: CUDA 13.3.1-devel-ubuntu22.04 base, pinned uv 0.12.5 / Python 3.13.15, full `requirements.lock` (196 pkgs). `ENTRYPOINT ["vllm", "serve"]` |
| `requirements.lock` | Frozen production Python stack — the single source of truth for the image's packages |
| `docker-compose.yml` | All services + vLLM serve flags (light setup: no MTP) — the canonical place serve behavior is configured |
| `docker-compose.mtp.yml` | Override file for heavy usage: full replacement of the vllm `command` list that adds the MTP speculative-decode flag, shipping the dedicated-card ideal-usage tuning (3 seq / 0.965, the shipped full-MTP tuning — bench runs single-stream); run `docker compose -f docker-compose.yml -f docker-compose.mtp.yml up -d` |
| `setup.sh` | Installs pinned `nvidia-container-toolkit==1.20.0-1`, creates `.venv_download/`, downloads weights into `./models/<MODEL_SUBDIR>/` via pinned `huggingface_hub[cli]==1.27.0` `hf download` |
| `.env.example` | Template for `.env`; `docker compose` **and** `setup.sh` both read it |
| `caddy/Caddyfile` | gateway on container port 8030: `/v1/*` (Basic → bearer translation, or the vLLM key passed through as-is for 8020 parity), `/metrics`, `/dcgm/metrics` (Basic only) |
| `prometheus/prometheus.yml` | Scrapes `vllm:8000/metrics` + `dcgm-exporter:9400` every 15 s, 30 d retention |
| `grafana/` | Auto-provisioned datasource + dashboard `vllm-qwen3827b.json` (folder **vllm**) |
| `dcgm-exporter/` | Minimal NVML→Prometheus sidecar (public base image + `nvidia-ml-py` — no NGC login needed). Swap for `image: nvcr.io/nvidia/dcgm-exporter:3.3.9-3.6.0-ubuntu22.04` if you have NGC access |
| `benchmarks/` | `context_bench.py` (TTFT/prefill/decode per context size) and `parallel_bench.py` (aggregate throughput vs concurrency); raw runs in `results/` |
| `models/qwen3.8-27b-nvfp4/` | Weights — **empty in git**, filled by `setup.sh` (gitignored, ~19 GB) |
| `.venv_download/`, `unsloth-nvfp4-env/`, `.venv/` | Local venvs, gitignored build artifacts |

## Services & ports

| Service | Host port | Notes |
|---|---|---|
| `vllm` | `8020` (→ container `8000`) | Direct full API. Auth: `VLLM_API_KEY` bearer on `/v1/*` — **empty key = fully open (LAN only)**. `/metrics`, `/health`, `/docs` stay open by vLLM design |
| `caddy` | `8030` (→ container `8030`) | gateway: Basic auth; the plain vLLM key is also accepted on `/v1/*`; fail-closed: **refuses to start if `METRICS_HASH` is unset** |
| `grafana` | `3000` (fixed, all interfaces) | Login `admin` / `GRAFANA_ADMIN_PASSWORD` (sign-ups disabled) |
| `prometheus` | `127.0.0.1:9090` (fixed, loopback) | Remote access via `ssh -L 9090:localhost:9090` |
| `dcgm-exporter` | none | Docker-network only (`dcgm-exporter:9400`); external reads via `8030/dcgm/metrics` |

All services run `restart: unless-stopped` — they come back after a crash, but stay stopped after a manual `docker stop`.

Port override keys in `.env`: `VLLM_HOST_PORT` (default 8020), `GATEWAY_HOST_PORT` (default 8030). **Container-side ports are fixed** (`vllm:8000`, `caddy:8030`) — only the host mapping changes. If you also want to move a container port, touch the `ports` block in `docker-compose.yml` *and* the site line in `caddy/Caddyfile` together.

## Configuration model

- **Model selection** — `.env` keys, all optional (unset = defaults for the stock Qwen3.8-27B NVFP4 model): `MODEL_REPO`, `MODEL_SUBDIR`, `SERVED_MODEL_NAME`, `CONTAINER_NAME`, `MODEL_REVISION` (pinned known-good sha; set empty to track latest).
- **Switching models**: set the keys → re-run `./setup.sh` → `docker compose up -d` (weights are a mount, the image is model-agnostic — no rebuild needed). Model-specific serve flags (`--quantization`, parsers, …) deliberately stay explicit in `docker-compose.yml`; for another model ship a full replacement `command` list in a second file and run `docker compose -f docker-compose.yml -f mymodel.yml up -d`.
- **vLLM serve flags** (in `docker-compose.yml`, don't scatter elsewhere): `--quantization modelopt`, `--kv-cache-dtype fp8`, `--max-model-len 262144`, `--max-num-seqs ${VLLM_MAX_NUM_SEQS:-16}`, `--gpu-memory-utilization ${VLLM_GPU_MEMORY_UTILIZATION:-0.95}`, `--reasoning-parser qwen3`, `--tool-call-parser qwen3_xml`, `--enable-auto-tool-choice`, `--enable-prefix-caching`, `--trust-remote-code` (required by the modelopt config), `--tensor-parallel-size 1`. The `docker-compose.mtp.yml` override adds `--speculative-config '{"method": "mtp", "num_speculative_tokens": 2}'` (MTP speculative decoding, draft length 2). Also env: `MAX_JOBS=4`, `NVCC_THREADS=4`, `shm_size: 16gb`.
- **Performance mode** — light setup (base file, no MTP): `.env` keys `VLLM_GPU_MEMORY_UTILIZATION` (base default `0.95`; sensible shared-host profile where the card may also drive the display / run other workloads) and `VLLM_MAX_NUM_SEQS` (base default `16`). Heavy usage = run the `docker-compose.mtp.yml` override (MTP-2), which ships the dedicated-card ideal-usage tuning (`3` seq / `0.965`, the shipped full-MTP tuning — bench runs single-stream, so the decode rate is unaffected by the sequence cap; assumes a dedicated card — no HDMI/DP, desktop on the iGPU, no other GPU workloads). The `.env` keys override the tuned defaults (e.g. the light values for a shared host) — documented in the README section "Usage modes".
- Served model name is `qwen3.8-27b` — this is what clients send in `model:`.

## Pinning discipline

Every input is pinned so rebuilds are reproducible. When updating a pin, follow the repo's own recipe (README "Pinned versions"): change the relevant line(s), rebuild, re-run the smoke test **and** a short benchmark, then commit with type `build`.

| Input | Where |
|---|---|
| vLLM stack (196 packages) | `requirements.lock` (regenerate: `docker exec <ctr> sh -c 'cd /app && uv pip freeze'`, keep `pkg==version` lines only) |
| CUDA base image, uv, Python | `Dockerfile` |
| HF download CLI, container toolkit | `setup.sh` |
| caddy / prometheus / grafana | images in `docker-compose.yml` |
| dcgm-exporter base + `nvidia-ml-py` | `dcgm-exporter/Dockerfile` |
| Model weights | `MODEL_REVISION` pin in `setup.sh` (override via `.env`) |
| Host driver (tested) | `610.43.02` (RTX 5090) |

## Commit conventions

Conventional Commits (see `CONTRIBUTING.md`, template in `.gitmessage`):
`<type>[scope]: <imperative subject ≤ 72 chars>` — types `feat fix docs style
refactor perf test build ci chore revert`. One logical change per commit;
`build` covers Dockerfile/lock/version changes. Suggested: `git config commit.template .gitmessage`.

## Gotchas (learned failures — check these first)

1. **First boot is slow, later boots fast**: FlashInfer JIT-compiles Blackwell FP4 GEMMs (`nvcc` in-container) and CUDA graph capture is RAM-hungry. "Compiling kernels…" is normal. On **OOM during graph capture** lower `MAX_JOBS`/`NVCC_THREADS` (e.g. `2`).
2. **64 GB host RAM can be marginal**; 128 GB recommended.
3. **caddy is fail-closed**: without `METRICS_HASH` in `.env` the 8030 gateway never starts (by design).
4. **bcrypt `METRICS_HASH` must be `$2a$`, not `$2b$`**: Python `bcrypt` emits `$2b$`, caddy/Go expects `$2a$` — use the one-liner in `.env.example` (it does the `replace()`).
5. **Grafana admin password is applied only on first DB init** (fresh `grafana_data` volume); later rotation is via the Grafana UI. Delete the volume to re-apply `.env` values.
6. **Benchmarks + prefix caching**: the server runs `--enable-prefix-caching`, so both suites use a *unique padding window per run* (token-accurate via `./models/*/tokenizer.json`, char-heuristic fallback) to keep prefills cold — don't "fix" the repeated-padding behavior, it's deliberate. A `--cached` run repeats a seed on purpose.
7. **Desktop hosts**: keep X/Wayland on an iGPU; the 5090 should be a pure compute card (GUI apps on the dGPU steal VRAM from the KV pool).
8. **Security posture**: never expose 8020/3000 unauthenticated; every 8030 route requires a credential — the Basic password on all routes, and `/v1/*` additionally accepts the plain vLLM key alone (the same credential 8020 requires). If you add a surface, document its auth in the README table.
9. **`setup.sh` is idempotent-ish but pinned**: re-running downloads the exact pinned revision; a missing/incomplete `models/` dir shows up as "model folder … does not exist" from vLLM.
10. **Keep README and code in lockstep**: port/flag/auth/serve-flag changes must update the README tables (Authentication & exposure, Configuration, Pinned versions, Project layout) in the same commit.
11. **caddy bakes `VLLM_API_KEY` at start**: the Caddyfile expands `{$VLLM_API_KEY}` when the caddy container starts, so its `/v1/*` routes pin the key it started with. A rotated key in `.env` only reaches caddy when its container is recreated (a plain `docker compose up -d` does that when caddy's env changes) — after a partial/manual restart, a 401 from 8030 `/v1` (even with correct Basic credentials) means caddy still holds the old key.

## Style

- Shell/Python/bash in this repo: comment *why* (known-good pins, failure modes), plain style, no frameworks.
- `docker-compose.yml` and the `Caddyfile` carry the security-related semantics in their header comments — preserve/extend those comments when editing.
- No unit tests; new tooling should stay dependency-light (stdlib + what `setup.sh` already installs).

---
> Source: [devbauerflorian/qwen3.8-27b-rtx5090](https://github.com/devbauerflorian/qwen3.8-27b-rtx5090) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
