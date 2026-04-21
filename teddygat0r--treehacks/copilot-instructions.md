## treehacks

> Distributed speculative decoding: a **draft node** (small model, local GPU) generates candidate tokens, a **target node** (large model, Modal cloud GPU) verifies them.

# Speculative Decoding System

## Architecture
Distributed speculative decoding: a **draft node** (small model, local GPU) generates candidate tokens, a **target node** (large model, Modal cloud GPU) verifies them.

### Key Components
- `workers/draft_node/client.py` — `DraftNodeClient`: generates draft tokens with vLLM, sends to target for verification. Supports multi-candidate speculative decoding (N candidates per round).
- `workers/target_node/server_modal.py` — `VerificationService`: Modal-deployed target model. Methods: `verify_draft` (single candidate), `verify_multi_candidate` (N candidates, single forward pass).
- `workers/target_node/server.py` — gRPC-based local target server (alternative to Modal).
- `workers/target_node/verification_strategies.py` — Pluggable verification strategies: `deterministic`, `probabilistic`, `threshold`, `greedy`. Supports `verify()` (single) and `verify_batch()` (vectorized N-candidate).
- `workers/benchmark_gsm8k.py` — Benchmark harness for GSM8K and HumanEval datasets.
- `proto/` — Protobuf definitions (`common.proto`, `speculative_decoding.proto`).

### Multi-Candidate Speculative Decoding
- Draft node generates N candidates per round using `SamplingParams(n=N)`.
- Target runs ONE forward pass for all candidates (they share the prefix).
- Best candidate selected by: longest accepted prefix, tie-break by target logprob sum.
- Adaptive N: reduces candidate count if acceptance rate stays below 20% over 5 rounds.
- CLI: `--num-candidates N`, `--candidate-temp`, `--candidate-top-p`.

## Models
- Draft: `Qwen/Qwen2.5-0.5B-Instruct` (default in benchmark) or `Qwen/Qwen3-1.7B-Instruct`
- Target: `Qwen/Qwen2.5-3B-Instruct` (Modal A100) or `Qwen/Qwen3-32B-Instruct`

## Commands
```bash
# Deploy target service to Modal
modal deploy workers/target_node/server_modal.py

# Run benchmark (single candidate)
python workers/benchmark_gsm8k.py --num-samples 5 --max-tokens 64

# Run benchmark (multi-candidate)
python workers/benchmark_gsm8k.py --num-samples 5 --max-tokens 64 --num-candidates 2

# Run local target server (gRPC, no Modal)
python workers/target_node/server.py --model Qwen/Qwen2.5-0.5B-Instruct --strategy deterministic

# Run draft client directly
python workers/draft_node/client.py
```

## Tech Stack
- Python 3.12, vLLM (inference), Modal (cloud GPU), gRPC + protobuf (local comms)
- Frontend: Next.js (in `app/`, `components/`)
- macOS ARM (Apple Silicon) for local dev; Modal A100 for target

## Conventions
- Activate venv before running: `source .venv/bin/activate`
- Proto stubs are pre-generated in `proto/` (no build step needed)
- Benchmark results saved to `workers/logs/` as JSON

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Teddygat0r) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
