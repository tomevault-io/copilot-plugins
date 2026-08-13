## ai-cluster

> This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Project Overview

AICluster is a distributed LLM inference platform with two core components:
- **Coordinator** (`coordinator/`) — Python FastAPI service providing an OpenAI-compatible REST API, worker discovery, load balancing, and model registry.
- **Worker** (`worker/`) — Rust service that runs GPU inference through two engines and exposes a gRPC endpoint: **llama.cpp/GGUF** (primary/recommended engine for consumer GPUs — native quantization, e.g. Q4_K_M/Q5_K_M/Q8_0; opt-in via the `llamacpp` cargo feature) and **Burn** (FP32 reference/experimental engine, safetensors; `wgpu` is the **default** cargo build feature — llama.cpp is not compiled unless requested).

Clients talk REST to the coordinator; the coordinator talks gRPC (protobuf) to workers. Protocol definitions live in `proto/cluster.proto` and generated bindings are in `coordinator/proto/` and built by `worker/build.rs`.

## Commands

### Docker Compose (full stack)
```bash
docker compose up -d --build   # Start coordinator + worker + Prometheus + Grafana + Open-WebUI
docker compose logs -f
docker compose down
```

### Coordinator (Python)
```bash
# Run from the REPO ROOT — `cd coordinator && uvicorn main:app` breaks package imports.
pip install -r coordinator/requirements-dev.txt   # runtime+lint+test (runtime only: requirements.txt)
uvicorn coordinator.main:app --reload --host 127.0.0.1 --port 8000
```
> Security note: the coordinator refuses to start with a non-loopback `--host`
> (e.g. `0.0.0.0`) unless `COORDINATOR_API_KEYS` is set — secure by default.
> For a LAN-reachable coordinator, set `COORDINATOR_API_KEYS` first
> (comma-separated secrets, e.g. `openssl rand -hex 32`) and pass
> `--host 0.0.0.0`. See `.env.example` and `docs/deployment.md`.

### Worker (Rust)
`wgpu` (Burn engine, FP32) is the **default** cargo feature; add `llamacpp` for the **recommended** GGUF/quantized-inference engine on consumer GPUs.
```bash
cd worker
cargo build --release --features wgpu    # Universal — Vulkan/DX12/Metal, auto-detects AMD/NVIDIA/Intel (default; Burn engine, FP32)
cargo build --release --features cuda    # NVIDIA base kernels (runtime backend type is still Wgpu — native wiring planned)
cargo build --release --features rocm    # AMD base kernels (runtime backend type is still Wgpu — native wiring planned)
cargo build --release --features metal   # macOS — Metal via wgpu
# llama.cpp engine (GGUF models, recommended for consumer-GPU quantized inference) — combine with a Burn backend feature:
cargo build --release --features wgpu,llamacpp                  # llama.cpp on CPU
cargo build --release --features wgpu,llamacpp,llamacpp-vulkan  # llama.cpp Vulkan offload
cargo build --release --features cuda,llamacpp,llamacpp-cuda    # llama.cpp CUDA offload
# Cross-node ggml-RPC roles (peer/lead), opt-in — no extra deps, spawns external binaries:
cargo build --release --features wgpu,llamacpp-rpc
./target/release/ai-worker --port 50051
```
> There is no CPU-only/ndarray build; a GPU (or Vulkan software rasterizer) is required.

### Tests
```bash
# Rust worker tests (must pass a backend feature — CI uses wgpu)
cd worker && cargo test --features wgpu
cargo test --features wgpu config::tests::test_name   # single test by path
# llama.cpp engine (compiles llama.cpp via cmake; needs cmake + libclang)
cargo check --features llamacpp
cargo test --features llamacpp llamacpp_engine                  # unit tests
cargo test --features llamacpp -- --ignored llamacpp            # e2e (network, ~1 MiB GGUF)
# Cross-node ggml-RPC roles (no extra deps — spawns external binaries, no GPU needed to test)
cargo test --features wgpu,llamacpp-rpc rpc_server

# Python unit tests (run from repo root)
pytest coordinator/
pytest coordinator/tests/test_router.py                # single file
pytest coordinator/tests/test_router.py::test_name -v  # single test

# Integration / client smoke tests (require a running coordinator + worker)
python tests/test_client.py
python tests/cluster_chat.py
```
> Note: `coordinator/tests/` contains the unit suite covering `models`, `config`, `router`, and coordinator error paths (run `pytest coordinator/ -q` for the current count). Rust unit tests live inline (`worker/src/config.rs`, `worker/src/gpu_manager.rs`, `worker/models/common.rs`).

### Linting
```bash
# Python: Black (line-length 100) + Ruff + MyPy strict
black --line-length 100 coordinator/
ruff check coordinator/
mypy coordinator/

# Rust (both enforced in CI)
cargo fmt -- --check
cargo clippy -p ai-worker --features wgpu -- -D warnings
```

## Architecture

```
Client (REST) → Coordinator (FastAPI) → Workers (Rust: llama.cpp + Burn) → GPU
                      │
              Prometheus / Grafana
```

**Coordinator modules** (`coordinator/`):
- `main.py` — FastAPI app entry point, lifespan, CORS, Prometheus ASGI mount
- `api.py` — FastAPI routes (`/v1/completions`, `/v1/chat/completions`, `/v1/models`, `/v1/workers`, `/health`, `/metrics`)
- `coordinator.py` — Core orchestration logic
- `router.py` — Wired into worker selection: `least_load`, `round_robin`, `random`, `affinity` (session-keyed, TTL), `power_of_two`; per-worker circuit breakers
- `discovery.py` — Worker discovery: static list only (mDNS/broadcast/Consul are planned; selecting them fails fast)
- `models.py` — Model registry and lifecycle
- `config.py` — `Settings` (pydantic-settings), reads `COORDINATOR_*` env vars / `.env` only (no YAML config exists)
- `monitoring.py` — Prometheus metrics definitions and helpers
- `auth.py` — Minimal opt-in API-key auth middleware for the HTTP surface (`COORDINATOR_API_KEYS`/`COORDINATOR_API_KEY_FILE`)
- `identity.py` — Key -> caller identity resolution (role, model scope) backing `auth.py`
- `audit.py` — Append-only single-line JSON audit log for management actions; never logs prompts, bodies, or key material
- `proxy.py` — Transparent HTTP proxy to worker-local `llama-server` instances for `engine == "llamaserver"` models
- `body_limit.py` — ASGI middleware enforcing a maximum HTTP request body size

**Worker modules** (`worker/src/`):
- `main.rs` — CLI entry point (clap), tokio runtime, gRPC server startup
- `worker.rs` — gRPC service handlers
- `gpu_manager.rs` — GPU detection and VRAM management
- `model_loader.rs` — Burn engine: safetensors loading as FP32 (quantization ≠ NONE is rejected; quantized inference for this path is planned — already available today via the llama.cpp engine below); resolves HF repo via `LoadModelRequest.model_path`
- `llamacpp_engine.rs` — **llama.cpp engine (primary/recommended for consumer GPUs)** for GGUF models (feature `llamacpp`, crate `llama-cpp-2 =0.1.150`); native quantization (Q4_K_M/Q5_K_M/Q8_0/…); implements `TextGeneration`
- `llamaserver_process.rs` — spawn/health/kill supervision for a `llama-server` child process (engine `"llamaserver"`); always compiled (no cargo feature, no `llama-cpp-2` dependency)
- `rpc_server_process.rs` (feature `llamacpp-rpc`) — spawn/health/kill supervision for `ggml-rpc-server` peer children (cross-node ggml-RPC `distributed_role = "rpc_server"`); health is a raw TCP connect, not HTTP
- `backend.rs` — `WorkerBackend` type alias (Wgpu; cuda/rocm features compile burn's native kernels but runtime selection is not wired — planned)
- `config.rs` — Worker config struct, reads `worker.toml`
- `error.rs` — Shared error types (`thiserror`)
- `metrics.rs` — Prometheus metrics definitions
- `parallelism.rs` — Tensor/pipeline/expert parallelism core functions; `AllReduce<B>` trait; standalone TP/PP functions compile and are correct but not yet wired to the gRPC service layer

**Configuration files** (`config/`):
- `worker.toml` — flat worker settings (ports, gpu_ids, concurrency, HF token fallback, llamacpp thread/gpu-layer defaults; unknown keys rejected)
- `models.toml` — Model registry: architectures, memory requirements, HuggingFace repo ids, per-model `engine` ("burn" | "llamacpp" | "llamaserver") + `[models.X.gguf]` source (`llamaserver` = agentic tool calling via a worker-supervised `llama-server` child process the coordinator proxies to; needs the `llama-server` binary on the worker host — see docs/deployment.md)
- `prometheus.yml` — Prometheus scrape targets
- `alerts.yml` — Prometheus alert rules (written against the real metric names)
(The coordinator has no config file — `COORDINATOR_*` env vars only.)

## Key Development Patterns

### Adding a new GGUF model (llama.cpp engine, recommended)
1. Add an entry to `config/models.toml` with `engine = "llamacpp"` and a `[models.X.gguf]` block (`repo_id`, `file`, `n_gpu_layers`, `n_ctx`) — no `[models.X.architecture]` block needed (read from GGUF metadata) and no conversion step required.
2. Build the worker with the engine enabled: `cargo build --release --features wgpu,llamacpp` (add `llamacpp-vulkan`/`llamacpp-cuda` for GPU offload).
3. Load via API: `POST /v1/models/load {"model_name": "your-model"}`

### Adding a new Burn/safetensors model (reference engine)
1. Convert weights: `python scripts/convert_model.py <hf-repo> --output ./models/`
2. Add entry to `config/models.toml` (architecture, memory, HF repo ID, quantization flags)
3. Load via API: `POST /v1/models/load {"model_name": "your-model"}`

### Changing the gRPC interface
1. Edit `proto/cluster.proto`
2. Regenerate Python bindings: `python -m grpc_tools.protoc -I./proto --python_out=./coordinator/proto --grpc_python_out=./coordinator/proto ./proto/cluster.proto`, then re-apply the package import: `sed -i 's/^import cluster_pb2 as cluster__pb2$/import coordinator.proto.cluster_pb2 as cluster__pb2/' coordinator/proto/cluster_pb2_grpc.py`
3. Rust bindings regenerate automatically via `worker/build.rs` on `cargo build`

### Environment variables (`.env` / Docker)
| Variable | Default | Purpose |
|---|---|---|
| `GPU_INDEX` | 0 | Which GPU device index (compose replicas offset ports by it) |
| `GPU_IDS` | — | Comma-separated device indices for one worker process (`--gpu-ids`) |
| `HF_TOKEN` | — | HuggingFace token for gated models (wins over worker.toml `hf_token`) |
| `RUST_LOG` | info | Worker log level (wins over `LOG_LEVEL`) |
| `LOG_LEVEL` / `LOG_JSON` | info / off | clap-level log settings |
| `RUST_BACKTRACE` | 1 | Rust panic backtrace (set to `full` for verbose) |
| `GPU_VRAM_GB` | — | Explicit memory override; wins over vendor detection |
| `GPU_MEMORY_HEADROOM_PERCENT` | 15 | Share of system RAM held back on unified-memory adapters |
| `WORKER_GRPC_BIND_HOST` | 127.0.0.1 | gRPC bind interface; compose sets `0.0.0.0` |
| `LLAMASERVER_BIND_HOST` | 127.0.0.1 | `llama-server --host`; compose sets `0.0.0.0` |
| `RPC_SERVER_BIND_HOST` | 127.0.0.1 | `ggml-rpc-server -H`; set to the interconnect address, never `0.0.0.0` (no auth/encryption) |
| `RPC_SERVER_BINARY_PATH` | `ggml-rpc-server` | Path to the peer-role binary (feature `llamacpp-rpc`) |
| `WORKER_GRPC_AUTH_TOKEN` | — | Shared secret required on every gRPC call when set |
| `WORKER_ID` | — | Unique worker identifier (auto-assigned if empty) |
| `GRPC_PORT` / `METRICS_PORT` | 50051 / 9091 | Explicit ports the binary reads (CLI/env > worker.toml > default) |
| `GRPC_BASE_PORT` / `METRICS_BASE_PORT` | 50051 / 9091 | Docker entrypoint only: replica port = base + GPU_INDEX (the bare binary ignores BASE vars) |

## CI / GitHub Actions

`.github/workflows/ci.yml` runs on every push/PR to `master` and `feature` branches:
- **Rust job** (`ubuntu-latest`): `cargo check` for `wgpu`, `llamacpp` (compiles the llama.cpp engine via cmake + libclang), `cuda`, `rocm` — the last two are type-check only, no CUDA/ROCm toolkit on the runner — plus `cargo fmt --check`, `cargo clippy -D warnings`, `cargo test --features wgpu`
- **Rust (aarch64) job** (`ubuntu-24.04-arm`): `cargo check` for `wgpu`, `llamacpp`, `cuda`; `rocm` is deliberately skipped here — cubecl-hip 0.8.1 has an aarch64-only `Vec<i8>`/`Vec<u8>` build bug
- **Rust (macOS/Metal) job** (`macos-latest`): `cargo check --features metal` — the only feature gated on `target_os = "macos"`, so it cannot be attempted on a Linux runner at all
- **Python job**: `ruff check`, `black --check`, `mypy` (strict; pydantic plugin; `coordinator/proto/` excluded), `pytest coordinator/`
- Not covered: `llamacpp-cuda`/`llamacpp-vulkan` (llama-cpp-sys-2 needs a real nvcc / Vulkan SDK build, not a toolchain-less type-check like the `cuda`/`rocm` features above)

## Worker Model Architecture

**`common.rs`**: `build_causal_bias<B>()` (O(seq²) once per prefill, shared by all model prefills), `RotaryEmbedding::apply()` (panic guard on bounds; cos/sin are [max_seq_len, head_dim/2]), `top_k_top_p_sample()` (real multinomial sampling via `rand::StdRng`, seedable from `InferenceRequest.seed`; temperature < 0.01 → greedy argmax in callers), `load_eos_ids()` (eos ids from (generation_)config.json), `swiglu()`, `repeat_kv()`.

**`mod.rs`**: `TextStream`, `TextGeneration` trait (`generate(..., seed: Option<u64>)`), `ModelInstance` (holds `Arc<Mutex<dyn TextGeneration>>`; increments `inference_count`; errors when no model is attached); re-exports `KvEntry<B>` / `KvCache<B>` from `llama.rs` for use by all model modules.

**`llama.rs`**: Reference implementation. `KvEntry<B>` = `(Tensor<B,4>, Tensor<B,4>)` per layer. `LlamaAttention::forward()` accepts pre-built `causal_bias`. `Llama::prefill()` → `(Vec<f32>, KvCache<B>)`; `decode_step()` O(seq_cached). `TextGeneration::generate()` — single `spawn_blocking` + mpsc channel, model cloned once.

**`qwen.rs`**: Qwen2/2.5 family — Llama-style GQA + RoPE + SwiGLU **plus** optional q/k/v biases (`attention_bias` from config.json) and explicit `head_dim` support. Qwen3 checkpoints are rejected at load (per-head q/k-norm unimplemented). Config is always built from the checkpoint's config.json. `QwenAttention` has `forward_prefill()` (returns `KvEntry`) and `forward_decode()`.

**`deepseek.rs`**: MoE with V1/V2-style sparse top-k routing (CPU sort → GPU weight broadcast; V3 sigmoid/group routing + MLA NOT implemented). `DeepSeekConfig` is built from the checkpoint's config.json (`n_routed_experts` supported); dense DeepSeek checkpoints (no `mlp.experts.*` keys) load through the Llama record path. EOS ids come from (generation_)config.json.

**`mistral.rs`**: Sliding window causal mask; query `i` attends to `[max(0,i-window+1), i]`.

**`model_loader.rs`**: Async safetensors load; spawn_blocking for dtype conversion (all weights land as FP32). Architectures: `"llama"`, `"qwen"`, `"deepseek"` (detected via `config.json` `"architectures"`). Downloads use `LoadModelRequest.model_path` (HF repo id resolved by the coordinator) with `model_name` as the registry key. Per-model loading lock + GPU reservation rollback on failure; `unload()` releases reservations. `create_deepseek_record()` loads `N` experts from `model.layers.{i}.mlp.experts.{j}.*` when present. Cross-node ggml-RPC routing (`distributed_role` metadata, feature `llamacpp-rpc`): `rpc_server` supervises one `ggml-rpc-server` child per GPU lent (`load_rpc_server_peer`), reserving `rpc_reserve_bytes` or, absent that, each GPU's full available memory; `lead` threads `rpc_peers`/`tensor_split` metadata into the spawned `llama-server`'s `--rpc`/`--tensor-split` flags before falling through to the normal `load_llamaserver_model` path.

**`llamacpp_engine.rs`** (feature `llamacpp`): `LlamaCppEngine::load(path, n_gpu_layers, n_ctx, n_threads)`; shared `Arc<LlamaModel>`, per-call `LlamaContext` inside one `spawn_blocking` + mpsc (same streaming shape as `llama.rs`); sampler chain from request temperature/top_k/top_p (greedy < 0.01), seedable from `InferenceRequest.seed`; EOS from GGUF metadata. Routing: the coordinator sends `engine`/`gguf_repo_id`/`gguf_file`/`n_gpu_layers`/`n_ctx` in the existing `ModelConfig.metadata` map (zero proto change); `model_loader.rs::gguf_spec_from_metadata` parses it and `load_llamacpp_model` downloads the GGUF via hf-hub, reporting the file size as memory.

**`gpu_manager.rs`**: O(1) memory tracking via `AtomicU64` with tagged `allocate_memory`/`free_memory`; telemetry (util/temp/power) refreshed from `nvidia-smi` at scrape/health time (3s timeout); CPU adapters dropped whenever a real GPU exists.

**`worker.rs`**: `active_requests` = `Arc<DashMap<String, Instant>>` (RAII `ActiveGuard` cleanup); `loaded_models` = `Arc<RwLock<HashMap<String, ModelInstance>>>`; `infer` bounded by a `max_concurrent_requests` semaphore (RESOURCE_EXHAUSTED beyond it); finish reasons: Stop / Length / Timeout / Error.

**`parallelism.rs`**: `TpKvCache<B>`, `AllReduce<B>` + `LocalAllReduce`. `tensor_parallel_llama_prefill/decode_step`, `pipeline_parallel_llama_forward`. `ParallelStrategy` enum (ExpertParallel stub). TP/PP standalone — not yet wired to gRPC.

## Git Conventions

When generating commit messages use Conventional Commits format (`feat`/`fix`/`chore`/`docs`) and reference the specific files changed. Keep the subject line under 72 characters. Always summarize the key changes across all modified files in the commit body.

## Docker & GPU

This project uses Docker with NVIDIA GPU support and Vulkan. Dockerfiles must include appropriate NVIDIA base images (`nvidia/cuda`) and Vulkan SDK layers (`libvulkan-dev`, `mesa-vulkan-drivers`). Always refer to existing Dockerfiles for patterns before creating new ones.

- `docker/Dockerfile.coordinator` — coordinator image (Python/FastAPI)
- `docker/Dockerfile.worker` — ONE parameterized worker image: default wgpu/Vulkan; `--build-arg BACKEND=rocm|cuda` with matching `BUILDER_IMAGE`/`RUNTIME_IMAGE`/`*_EXTRA_PKGS` args for the vendor variants (see the file header and docker-compose.yml comments). Also bakes in a `llama-server` binary (Vulkan, from a pinned llama.cpp tag) for `engine = "llamaserver"` agentic models and sets `LLAMASERVER_BINARY_PATH`; opt out with `--build-arg LLAMASERVER_SRC=llamaserver-none`. See docs/deployment.md "llama-server for agentic serving". Does NOT bake in `ggml-rpc-server` (cross-node `rpc_server` peer role) — install it separately and set `RPC_SERVER_BINARY_PATH`, see docs/deployment.md "Cross-node ggml-RPC split".
- AMD passthrough: mount `/dev/kfd` + `/dev/dri`, add `group_add: [video, render]`
- NVIDIA passthrough: use `deploy.resources.reservations.devices` (NVIDIA Container Toolkit)
- Intel GPU works out of the box with `Dockerfile.worker` via Mesa Intel ANV Vulkan driver
- When modifying Dockerfiles, ensure GPU passthrough and Vulkan layers are preserved
- Always update `.env.example` when adding new environment variables to docker-compose or config
- GPU setup helper scripts: `scripts/setup_cuda.sh` (NVIDIA toolkit) and `scripts/setup_rocm.sh` (AMD ROCm)

## Languages & Build

Primary languages: Python (coordinator), Rust (worker), YAML (configs/CI), Shell scripts, Markdown docs.

- Always use Python type hints; keep YAML files consistent with existing formatting and indentation
- After modifying Rust files: `cd worker && cargo check`
- After modifying Python files: `python -m py_compile <file>`
- After modifying proto files: regenerate bindings (see "Changing the gRPC interface" above)

---
> Source: [caestrada1103/ai-cluster](https://github.com/caestrada1103/ai-cluster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
