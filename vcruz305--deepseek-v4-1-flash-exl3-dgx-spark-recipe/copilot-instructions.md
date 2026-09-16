## deepseek-v4-1-flash-exl3-dgx-spark-recipe

> Guidance for automated agents and contributors working in this recipe repository.

# AGENTS.md

Guidance for automated agents and contributors working in this recipe repository.

## Scope

This repo is a **serving recipe**, not the EXL3 quantizer and not a fork of DeepSeek/vLLM. Keep model architecture changes in upstream/fork repos and EXL3 plugin changes in `vcruz305/vllm-exl3`.

## Non-negotiable rules

1. Preserve both TP2 and TP4 recipes. Do not optimize one by silently breaking the other.
2. Never replace the dedicated DeepSeek V4.1 base image with stock pip vLLM. The V4.1 architecture is image-pinned.
3. Do not silently advance `vllm-exl3`, ExLlamaV3, the base image, CUDA flags or a model revision. Update the pin table and qualification notes together.
4. TP4+EP4 is the correctness-first path: 96 whole main experts/rank at 5120 x 2304.
5. TP2+EP2 is experimental: 192 whole main experts/rank and a much tighter memory budget.
6. The ABI-3 V4.1 native MoE path must remain opt-in until hardware evidence supports changing that policy.
7. Do not claim skipped CUDA tests as passes.
8. Do not publish throughput numbers without recording exact runtime identity, model revision, topology, context, batch, speculative policy and actual EXL3 backend.
9. Keep Engram variants explicit. A disk-backed or node-local Engram patch is a separate experimental variable, not a hidden part of the baseline.
10. Preserve third-party attribution. If code is copied/adapted, add exact source URLs/commits/files and licensing to `THIRD_PARTY_NOTICES.md`.

## Validation before merging

At minimum:

```bash
bash -n scripts/*.sh
python -m py_compile scripts/preflight.py
python -m json.tool configs/quantization_config.example.json >/dev/null
```

For runtime changes, test the Docker build on a DGX Spark and run:

```bash
./scripts/preflight.sh 4
./scripts/runtime_identity.sh
```

A runtime change touching TP2 must also run the TP2 preflight and document whether a real TP2 pack was loaded.

## Benchmark discipline

Keep these separate:

- kernel microbenchmark;
- full-model decode speed;
- prefill throughput / TTFT;
- aggregate multi-request throughput;
- DSpark acceptance and speedup;
- long-context memory behavior.

Do not combine warm prefix-cache hits with cold TTFT measurements.

---
> Source: [vcruz305/DeepSeek-V4.1-Flash-EXL3-DGX-Spark-recipe](https://github.com/vcruz305/DeepSeek-V4.1-Flash-EXL3-DGX-Spark-recipe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
