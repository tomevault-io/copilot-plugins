## mlx-tinker

> Tinker API-compatible backend for Mac (Apple Silicon / Metal) using MLX and mlx-lm.

# MLX-Tinker

Tinker API-compatible backend for Mac (Apple Silicon / Metal) using MLX and mlx-lm.

## Framework scope
- **Mac / Apple Silicon / Metal only** — no CUDA, no multi-node
- **Heterogeneous compute** — utilize both CPU and GPU where possible (e.g. CPU for data prep/tokenization, GPU for forward/backward)
- **Single-node, single-process** — unified memory, no distributed workers
- **Optimized for extremely fast SFT and RL** on local hardware
- **Speed and memory over numerical exactness** — aggressive quantization (4-bit, dynamic tree quant, chunked CE) is encouraged when it saves memory or wall-clock time; bit-exact parity with cloud Tinker is a non-goal
- Target use case: local AI agents on Apple Silicon that need fast continual learning (SFT, PPO, CISPO, DRO)

## Project overview
- Three-layer architecture: API (FastAPI) → Engine (asyncio) → Backend (MLX)
- v0.1 targets Qwen3.5-9B with QLoRA finetuning
- Single-process, unified memory — no distributed workers

## Commands
- Install: `uv sync --all-extras`
- Run server: `uv run python -m mlx_tinker --model Qwen/Qwen3.5-0.8B`
- Tests: `uv run pytest tests/ -k "not stress and not cookbook"`
- Stress tests: `uv run pytest tests/stress/ -m stress`
- Cookbook tests: `uv run pytest tests/cookbook/ -m cookbook`
- Lint: `uv run ruff check mlx_tinker/`
- Format: `uv run ruff format mlx_tinker/`

## Testing requirements
- ALWAYS run unit tests, cookbook tests, AND stress tests before committing
- All three test suites must pass (stress tests may have infra issues like deprecated datasets — those are pre-existing)
- Run: `uv run pytest tests/ -k "not stress and not cookbook" && uv run pytest tests/cookbook/ -m cookbook`

## Architecture
- `mlx_tinker/api/` — FastAPI server, Tinker endpoints + OpenAI-compat
- `mlx_tinker/engine/` — Async polling loop with barrier-aware batching
- `mlx_tinker/backend/` — MLX compute: training, inference, QLoRA, checkpointing
- `mlx_tinker/db/` — SQLModel ORM with async SQLite (WAL mode)
- `mlx_tinker/types.py` — Shared enums and Tinker-compatible types
- `mlx_tinker/config.py` — Pydantic config

## Key patterns
- All training/sample endpoints return FutureResponse; client polls retrieve_future
- Engine runs 100ms cycles, batches compatible requests, respects barriers (optim_step blocks)
- Backend uses nn.value_and_grad for forward_backward, accumulates grads until optim_step
- QLoRA: 4-bit quantized base + LoRA adapters via mlx-lm tuner utilities

## Design decisions — Tinker API parity

Goal: 
1. `tinker.ServiceClient(base_url="http://localhost:8000", api_key="tml-local")` and all tinker-cookbook code must work against mlx-tinker without modification.
2. mlx-tinker can be used with ANY downstream application that utilizes tinker API (only Qwen3.5 family supported for now).
3. Extremely fast, performant model training + inference platform for local AI agents on Apple Silicon.
4. Integrate with Hermes Agent and OpenClaw to support continual learning plugins such as OpenClaw-RL.

## Development Principles

- All claimed optimizations must have proofs with them.
- As a faithful AI agent building a high-stakes, upstream library, you are required to be extremely thorough.
- ALWAYS VERIFY CLAIMS AND ASSUMPTIONS.
- Do experiments, A lot of them. Verify ideas in small scale. Write one-off scripts if required. Only high-confidence changes will be approved.
- The goal for `mlx-tinker` is to help power "local AI agents that learn with you". 
- OpenClaw is the agent harness, OpenClaw-RL is the learning layer, mlx-tinker is the compute layer. 
- OpenClaw-RL already has support for tinker backend, point tinker's base url to mlx-tinker deployment and everything should just work.

### Loss functions
- **Sum reduction** for all losses (not mean) — matches Tinker's official formulas
- `cross_entropy`: `(-target_logprobs * weights).sum()`
- `importance_sampling`: `-(ratio * advantages).sum()` — advantages=0 at prompt positions is the implicit mask; no separate mask/weights field
- `ppo`: `-(min(ratio * adv, clip(ratio) * adv)).sum()`
- `cispo`: `-(sg(clip(ratio)) * target_logprobs * advantages).sum()` — stop-gradient on clipped ratio, not min-of-two like PPO
- `dro`: `-(target_logprobs * advantages - 0.5 * beta * (target_logprobs - sampling_logprobs)^2).sum()`

### Datum and LossFnInputs
- `loss_fn_inputs` is a flexible dict (`Dict[str, TensorData]`), not a fixed struct
- Only `target_tokens` is required; `weights`, `advantages`, `logprobs` are optional
- Defaults when missing: `weights` → all-ones, `advantages` → all-zeros, `logprobs` → all-zeros
- `TensorData` includes `data`, `dtype` (`"float32"` | `"int64"`), and optional `shape`

### Wire format
- Requests use nested keys: `forward_backward_input` / `forward_input` (not flat)
- All requests/responses carry `type` discriminator literals (e.g. `"create_model"`, `"optim_step"`, `"sample"`)
- Submit endpoints return `{"request_id": "..."}` (UntypedAPIFuture)
- `retrieve_future` returns the raw typed result when complete, or `{"type": "try_again", "request_id": "...", "queue_state": "active"}` when pending — no wrapper object

### Metrics and outputs
- Primary metric key is `loss:sum` (not `mean_loss`)
- `ForwardBackwardOutput.loss_fn_outputs` is `List[Dict[str, TensorData]]` where each dict has a `"logprobs"` key with per-token log-probabilities (not `{"loss": float}`)

---
> Source: [prem-research/mlx-tinker](https://github.com/prem-research/mlx-tinker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
