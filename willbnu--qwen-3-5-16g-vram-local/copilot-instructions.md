## qwen-3-5-16g-vram-local

> Gemini-facing project context for `qwen-llm`.

# GEMINI.md

<<<<<<< HEAD
Gemini-facing project context for `qwen-llm`.

## Repository purpose

This repo packages tested local serving and benchmarking workflows for Qwen3.5 GGUF models on a 16GB GPU, with an emphasis on reproducible launcher settings and evidence-backed performance notes.

## Important repo truths

- Config source of truth: [config/servers.yaml](config/servers.yaml)
- Runtime launcher: [server_manager.py](server_manager.py)
- Benchmark summaries should trace back to JSON files in `results/`
- `personal/dev` may intentionally differ from `origin/main`

## Model-family projector policy

Use distinct local filenames:

- `mmproj-35B-F16.gguf`
- `mmproj-27B-F16.gguf`
- `mmproj-9B-F16.gguf`

Do not unify them in the shared local model directory.

## Suggested validation path

1. Config regression tests
2. Start one preset
3. Health check
4. Text completion
5. Vision completion if applicable
6. Short benchmark pass
=======
Use [AGENTS.md](AGENTS.md) as the shared repo policy.

Additional note for Gemini-style collaborators:

- keep public docs aligned with the shipped presets in `config/servers.yaml`
- keep branch-local or maintainer-only notes out of release-facing docs
>>>>>>> origin/main

---
> Source: [willbnu/Qwen-3.5-16G-Vram-Local](https://github.com/willbnu/Qwen-3.5-16G-Vram-Local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
