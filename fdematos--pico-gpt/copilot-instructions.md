## pico-gpt

> Compact, repo-specific guidance. Read this **and** `PLAN.md` before doing anything.

# AGENTS.md — pico-gpt

Compact, repo-specific guidance. Read this **and** `PLAN.md` before doing anything.

## What this repo is

A step-by-step educational rebuild of [karpathy/nanoGPT](https://github.com/karpathy/nanoGPT) → [karpathy/nanochat](https://github.com/karpathy/nanochat). Goal is **understanding**, not beating any leaderboard. Each phase is one (or a few) commits. See `PLAN.md` for the phase list and exit criteria — **always** check which phase we are in before writing code: do not pull in features from later phases.

## State of the repo right now

The repository is **bootstrapping** (only `PLAN.md` + this file exist on disk so far). Nothing is implemented yet. When the first agent starts coding, follow `PLAN.md` Phase 1 to the letter; do not try to scaffold all phases at once.

## Hardware constraints — DO NOT FORGET

Target machine: **2× NVIDIA RTX 3090 (24 GB, SM 86, Ampere)**.

This rules out a number of things an agent will instinctively reach for from the nanochat code:

- **No FP8.** SM 86 has no fp8 tensor cores. Never copy `--fp8` flags or `torchao.float8`.
- **No Flash Attention 3.** FA3 is Hopper-only. Use `torch.nn.functional.scaled_dot_product_attention` (SDPA) or FA2 if installed. Never gate code behind `USE_FA3`.
- **bf16 is fine** (Ampere has bf16 tensor cores). Default `COMPUTE_DTYPE` should be `bfloat16` on CUDA, `float32` on CPU/MPS. fp16 + GradScaler path is **not** needed for our hardware — skip it.
- **DDP via `torchrun --nproc_per_node=2`**, never 8. Anything assuming 8 GPUs (batch size math, sharding) must be parameterised, not hard-coded.
- **24 GB VRAM per card.** A d=24 GPT-2-scale model with value embeddings will not fit comfortably; cap experiments at d≤16 unless explicitly reducing batch size with gradient accumulation. Always make `--device-batch-size` overridable.
- Everything must also degrade gracefully to **CPU / MPS** for laptop iteration (smaller `--depth`, fewer steps). Test that path on every change.

## Conventions

- **Language: English everywhere** — code, identifiers, comments (when unavoidable), commit messages, `dev/LOG.md`, this file.
- **Single dial of complexity is `--depth`.** Like nanochat, all other model hyperparameters (width, heads, head_dim, LR scaling, weight decay, batch size, training horizon) are derived from depth. Never add a CLI flag whose value could be computed from `depth`.
- **One feature per commit** during Phase 3+. The commit message must say which `dev/LOG.md` entry documents it.
- **No magic dtype.** Do not use `torch.amp.autocast`. Follow nanochat: a single `COMPUTE_DTYPE` global in `pico_gpt/common.py`, with a custom `Linear` that casts weights to input dtype in forward. Embeddings stored directly in `COMPUTE_DTYPE`. Override via `PICO_GPT_DTYPE` env var.
- **No silent failures.** Validate args at entry. If a CLI flag is incompatible with the hardware (e.g. `--fp8` on 3090), error out loudly with an explanation; never silently fall back.

## Toolchain

- **uv** is the dependency manager (matches upstream nanochat). Use:
  ```bash
  uv sync --extra gpu --group dev      # CUDA on the 3090 box
  uv sync --extra cpu --group dev      # laptop
  source .venv/bin/activate
  ```
- **`pyproject.toml`** must declare two explicit torch indexes (cpu vs cu12x wheel) under `[tool.uv.sources]`, same pattern as nanochat. `cpu` and `gpu` extras must be declared `conflicts` in `[tool.uv]` so a user can't activate both.
- **Tests**: `pytest`. Mark slow / GPU-only tests with `@pytest.mark.slow` and `@pytest.mark.gpu`. CPU-only smoke tests must run in under 60 s without CUDA.
- **No `pip install`, no `conda`**, no `requirements.txt`. Anything not in `pyproject.toml` does not exist.

## Project layout (target, populated phase by phase)

```
pico_gpt/        # the library (model, optim, dataloader, tokenizer, engine, common)
scripts/         # CLIs: tok_train, base_train, base_eval, chat_sft, chat_cli, chat_web
tasks/           # eval/SFT tasks: mmlu, gsm8k, humaneval, smoltalk, ...
data/            # data prep scripts; downloaded shards live in base_data*/ (gitignored)
tests/           # pytest suite
runs/            # bash entrypoints (runcpu.sh, run2x3090.sh)
dev/
  LOG.md         # dated experiment log — append, never rewrite history
  PLAN.md        # the phase plan (or kept at repo root, fine either way)
```

Do not create this whole tree up-front. Create directories as their phase begins.

## Workflow rules an agent will get wrong without this

- Before adding any architectural tweak, **check `dev/LOG.md`**: if it's marked as a negative result upstream (DyT, MoE, varlen, MTP, half-truncated RoPE, asymmetric softcap, FineWeb non-EDU, etc.), do not re-add it without a written reason.
- When in doubt about an upstream design choice, the source of truth ordering is: **`nanochat/*.py` code > nanochat `dev/LOG.md` > discussion #481 > README > other discussions**. Verify before quoting.
- Track every run with a one-paragraph entry in `dev/LOG.md` (date, depth, hardware, val bpb / loss, throughput, what changed, A/B vs previous). Negative results are required to be logged too.
- Datasets: never commit `*.bin`, `*.parquet`, model checkpoints, `wandb/`, or any of `base_data*/`. Update `.gitignore` before the first data prep script lands.
- Networking: pretraining data download will hit HuggingFace. Cache to `~/.cache/huggingface` (default) and document the env vars (`HF_HOME`, `HF_DATASETS_CACHE`) — do not invent custom env vars.

## When the user asks for "the next step"

1. Read `PLAN.md` and identify the current phase by looking at the last commit and the tree.
2. Stay strictly within that phase. If the phase is done, propose moving to the next phase and wait for confirmation before bringing in its new dependencies.
3. Always finish with: tests pass, smoke run executes, `dev/LOG.md` updated, then commit.

## References worth re-reading before each phase

- `https://github.com/karpathy/nanoGPT` (phase 1 only)
- `https://github.com/karpathy/nanochat` (everything from phase 2 onward)
- `https://github.com/karpathy/nanochat/discussions/481` — architectural rationale
- `https://github.com/karpathy/nanochat/discussions/420` — miniseries / depth as the dial
- `https://github.com/karpathy/nanochat/discussions/164` — adding abilities via SFT tasks
- `https://github.com/karpathy/nanochat/discussions/139` — infusing identity through SFT data mixing

Anything not covered here: ask first.

---
> Source: [fdematos/pico-gpt](https://github.com/fdematos/pico-gpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
