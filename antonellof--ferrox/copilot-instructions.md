## ferrox

> Guidance for agents working in this repo.

# CLAUDE.md

Guidance for agents working in this repo.

## What this is

Pure-Rust GGUF / MoE inference engine: mmap loaders, quantized CPU +
Metal + CUDA kernels, OpenAI-compatible `ferrox-server`.

**Works today (evidence in docs):**

- Dense GQA: TinyLlama, Llama-3.1-8B Q4_K_M (Metal fair-chat Gap
  **~0.97×**; CLI pin **~1.00×** — see `benchmarks/RESULTS.md`).
- Phi-4-mini Q4_K_M Metal pin (~1.06×); Gemma-4-E2B fail-closed until
  dedicated engine (per-layer emb / shared KV / SWA split).
- MoE GQA: OLMoE-1B-7B-0924 (CPU + Metal expert placement).
- Metal dense stack (`FERROX_METAL` / `FERROX_METAL_ATTN`); FA-vec
  decode d=64/96/128/256 + prefill d=128; CUDA matvec + resident weights
  + FFN fuse (CUDA fair-chat pins need a GPU host).

**Not e2e yet:** Gemma-4-E2B, real GLM-5.2 / DeepSeek V4 checkpoints;
full Kimi K3 (~1.56 TB). See `docs/MODELS.md`.

**Discipline:** evidence-first. Presets list `best_effort_fields`.
Authoritative docs:

| Doc | Role |
|---|---|
| `docs/CLI.md` | `ferrox` llama.cpp-style flags + `ferrox chat` |
| `docs/MODELS.md` | what runs / what doesn’t |
| `docs/API.md` | OpenAI compatibility matrix |
| `docs/AGENTS_COOKBOOK.md` | point IDEs at `ferrox-server` |
| `docs/CONFIG.md` | env vars (`FERROX_CTK`, Metal, CB, …) |
| `benchmarks/RESULTS.md` | tok/s vs llama.cpp (Gap = llama/ferrox) |
| `docs/ROADMAP.md` | future work only |

## Commands

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all

cargo build --workspace --features cuda
cargo test -p ferrox-cuda --features cuda -- --ignored   # needs GPU

cargo build -p ferrox-cli -p ferrox-server --features metal
cargo test -p ferrox-metal --features metal -- --ignored   # needs Metal

# Completion (also: ferrox run -m …)
./target/debug/ferrox -m model.gguf -p "Hi" -n 64 --temp 0 --no-cnv
./target/debug/ferrox -m model.gguf -p "Hi" -n 64 --ngl 99   # Metal

./target/debug/ferrox presets | archs | caps | inspect <gguf> | inspect-plan <gguf>
./target/debug/ferrox smoke <preset> | run-kimi <dir>
./target/debug/ferrox chat --url http://127.0.0.1:8383   # needs ferrox-server

FERROX_MODEL_PATH=model.gguf FERROX_ADDR=127.0.0.1:8383 ./target/debug/ferrox-server

# Fair-chat vs llama.cpp (Host B): see benchmarks/README.md
python3 benchmarks/run_suite.py --list
python3 benchmarks/run_suite.py --id llama31_8b_q4km --backend metal
# CUDA host (requires --features cuda binary + GPU):
python3 benchmarks/run_suite.py --id llama31_8b_q4km --backend cuda \
  --host-label "host / GPU / driver"
```

Fixtures and golden values were generated and cross-validated with
independent NumPy references.

Tests are mostly `#[cfg(test)]` next to the code. Integration:
`crates/ferrox-models/tests/gguf_roundtrip.rs`. Never un-ignore CUDA /
Metal hardware tests without a real GPU.

## Architecture

```
ferrox-gguf + ferrox-quant
        → ferrox-core (WeightMatrix, RoPE, GQA, KV; optional cuda/metal)
        → ferrox-moe
        → ferrox-models (loader, Decoder, Kimi/GLM/DS4 stacks)
        → ferrox-cli / ferrox-server
```

Load path: GGUF mmap → keep quantized → fused dequant+dot →
RMSNorm → GQA(+RoPE) → MoE/dense FFN. Serving: `FERROX_MODEL_PATH`
GGUF or Kimi dir; generation on `spawn_blocking`.

Presets `glm_5_2` / `deepseek_v4_pro` / `kimi_k3` are sketches —
not proof of real-checkpoint support. `test_*_fixture` presets match
Python test GGUFs only.

---
> Source: [antonellof/ferrox](https://github.com/antonellof/ferrox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
