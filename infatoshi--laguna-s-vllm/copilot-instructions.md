## laguna-s-vllm

> Home: Anvil-primary - canonical at anvil:/home/infatoshi/kernels/laguna-s-vllm. Run GPU work there via `ssh anvil`; the Mac is a client only.

Home: Anvil-primary - canonical at anvil:/home/infatoshi/kernels/laguna-s-vllm. Run GPU work there via `ssh anvil`; the Mac is a client only.

# Commands and constraints

- Before GPU work, run `nvidia-smi`; the tested device is GPU0, RTX PRO 6000 Blackwell 96 GB.
- Setup: `./scripts/setup.sh`.
- CPU-only validation: `./scripts/setup.sh --check`, `./scripts/serve.sh --help`, and `./scripts/claude-laguna --help`.
- Serve: `./scripts/serve.sh`. Default is concurrency one, DFlash K=3, 262,144 context, and vLLM 0.25.1.
- Before any speculative-decoding measurement, follow [BENCHMARK.md](BENCHMARK.md). Do not compare runs with different prompts, token lengths, sampling, concurrency, warmups, or GPU occupancy.
- The patch is valid only for vLLM 0.25.1 and is guarded to the safe single-request FlashInfer path.
- Validate changed shell scripts in this order: `shfmt -w scripts/*.sh scripts/claude-laguna`, `bash -n scripts/*.sh scripts/claude-laguna`, then `shellcheck scripts/*.sh scripts/claude-laguna`.
- Never configure Laguna through shared Claude Code settings. Use `scripts/claude-laguna`, which scopes the gateway variables to one process.

---
> Source: [Infatoshi/laguna-s-vllm](https://github.com/Infatoshi/laguna-s-vllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
