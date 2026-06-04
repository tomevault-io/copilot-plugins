## advanced-gguf-quantizer

> Public operating guide for AI agents and automated coding assistants working in

# AGENTS.md

Public operating guide for AI agents and automated coding assistants working in
`advanced-gguf-quantizer`.

This repository is a llama.cpp-derived GGUF quantization toolkit focused on
NVFP4, MXFP6_E2M3, mixed NVFP4/MXFP6 models, calibration, evaluation,
repair/edit workflows, and release-quality artifacts.

## Read First

- Start with [README.md](README.md) for the human workflow.
- Use [docs/README.md](docs/README.md) as the detailed documentation index.
- Use [SKILLS.md](SKILLS.md) for agent-driven quantization runs.
- Use [docs/advanced-gguf-quantizer-flags.md](docs/advanced-gguf-quantizer-flags.md)
  for recipe and CLI details.
- Use [docs/advanced-gguf-quantizer-layer-policy.md](docs/advanced-gguf-quantizer-layer-policy.md)
  for tensor grouping and output/head/MTP policy.
- Use [docs/advanced-gguf-quantizer-cuda-acceleration.md](docs/advanced-gguf-quantizer-cuda-acceleration.md)
  before changing CUDA or runtime patch evaluation behavior.
- Use [docs/advanced-gguf-quantizer-imatrix-kld.md](docs/advanced-gguf-quantizer-imatrix-kld.md)
  before changing imatrix, saved-logit KLD, or evidence reuse behavior.
- Use [docs/advanced-gguf-quantizer-nvfp4-contract.md](docs/advanced-gguf-quantizer-nvfp4-contract.md)
  before changing NVFP4 GGUF writer, loader, or scale tensor behavior.
- Preserve [NOTICE.md](NOTICE.md), including Michael Wand attribution and MIT
  HAN Lab Four Over Six attribution.

## Project Boundaries

This project quantizes local GGUF files. It does not download models, import
Hugging Face checkpoints, or convert source checkpoints. Users should create
source GGUFs with a separate llama.cpp checkout and use this repository for
advanced quantization and evidence.

Keep the retained public tool surface focused on:

- `llama-quantize`
- `advanced-gguf-quantizer`
- `llama-imatrix`
- `llama-perplexity`
- `llama-completion`
- optional `llama-bench`
- optional `llama-fit-params`

Do not add server, web UI, model download, checkpoint import, or unrelated app
features unless the maintainer explicitly asks for them.

## Development Principles

- Preserve llama.cpp style and upstream-compatible behavior where possible.
- Keep changes scoped and reviewable.
- Prefer recipe/project fields over environment-variable product controls.
- Do not add compatibility aliases for old private commands or local research
  paths.
- Do not remove advanced quantization features to make code smaller unless the
  feature is genuinely dead or duplicated.
- Preserve MTP/NextN metadata and tensors by default.
- Keep output/head policy explicit and conservative.
- Treat MXFP6_E2M3 as experimental and unsupported by NVIDIA and llama.cpp; do
  not remove the compatibility warning from docs or the TUI.
- Treat NVFP4 as the speed-first format. Quality, correctness, and
  reproducibility still require full evidence; speed claims require separate,
  quiet-GPU benchmarks rather than assumptions.
- Make frequent commits for meaningful checkpoints when operating as an agent.

## Build

Use an existing `build` directory when present:

```bash
cd build
ninja -j 20
```

For a fresh local build:

```bash
cmake -S . -B build -DGGML_CUDA=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j 20
```

Optional retained helper tools:

```bash
cmake -S . -B build \
  -DGGML_CUDA=ON \
  -DADVANCED_GGUF_QUANTIZER_BUILD_BENCH=ON \
  -DADVANCED_GGUF_QUANTIZER_BUILD_FIT_PARAMS=ON \
  -DCMAKE_BUILD_TYPE=Release
cmake --build build -j 20
```

## Verification

After C++ or CMake edits, run:

```bash
cd build
ninja -j 20
```

For CLI surface checks:

```bash
./build/bin/advanced-gguf-quantizer --help
./build/bin/llama-quantize --help
./build/bin/llama-imatrix --help
./build/bin/llama-perplexity --help
./build/bin/llama-completion --help
```

For text UI changes, run the PTY capture smoke:

```bash
python3 tools/quantize/advanced/capture_advanced_gguf_quantizer_tui.py \
  --out /tmp/agq-tui-capture \
  --rows 24 \
  --cols 100 \
  ./build/bin/advanced-gguf-quantizer
```

When tests are configured in a build tree, run the relevant `ctest` subset or
the full suite if the change touches shared runtime behavior.

## Quantization Workflow Expectations

Use recipes and projects for serious work:

```bash
./build/bin/advanced-gguf-quantizer recipe init --profile nvfp4_mxfp6 --output recipe.toml
./build/bin/advanced-gguf-quantizer recipe validate recipe.toml
./build/bin/advanced-gguf-quantizer plan recipe.toml
./build/bin/advanced-gguf-quantizer run recipe.toml --project runs/model --yes
```

Use `plan` and `recipe validate` for inspection. Use `run` when the user wants a
real model artifact. Do not use fake quantization passes as a substitute for a
real artifact run.

For saved-logit KLD base generation, use
`advanced-gguf-quantizer kld-command` and run the printed command as-is. It
should only name the reference GGUF, evaluation corpus, and output KLD base; do
not add runtime-shape or scheduling overrides.

Every production run should preserve:

- source GGUF path and hash when available;
- locked recipe;
- calibration corpus and imatrix;
- evaluation corpus and KLD base when used;
- final GGUF;
- tensor assignment log;
- run manifest;
- `checkpoint-key.json`;
- quantization report;
- validation smoke script.

## NVFP4 Candidate Policy

RSF is part of the normal NVFP4 candidate path by default. Do not hide RSF or
other product behavior behind environment variables; expose it through recipes,
project fields, or CLI flags.

Speed-aware tensor type candidates are also part of the main selector path, not
a post-hoc repair-only workflow. The selector should first try NVFP4/RSF
tensor-local repair; only tensors where NVFP4 still has high local error should
move to fallback types.

Useful flags:

- `--nvfp4-selector-candidate-types Q4_K,Q6_K,Q8_0`
- `--nvfp4-selector-candidate-fraction F`
- `--nvfp4-selector-candidate-top N`
- `--nvfp4-selector-candidate-budget-mb N`
- `--nvfp4-selector-candidate-class-limit N`
- `--nvfp4-selector-candidate-report file.csv`
- `--nvfp4-selector-candidate-tensor-types file.txt`

With `--nvfp4-fast-quantize`, the default cap is the worst 10% of NVFP4-error
candidate tensors. Mixed NVFP4/MXFP6 runs should prefer
`MXFP6_E2M3,Q4_K,Q6_K,Q8_0` so MXFP6 can win before Q4_K when the mixed format
is available. Do not run `llama-bench` inside quantization; benchmark final
artifacts separately on a quiet GPU.

## Quality Evidence

Do not claim quality from a proxy score, a first chunk, a tiny diagnostic sample,
or mismatched baselines.

Useful quality evidence includes:

- PPL;
- mean KLD;
- p95/p99/p999 KLD;
- KLD tail mean and max KLD;
- RMS probability delta;
- same-top rate;
- top-flip weight;
- entropy drift;
- non-finite counts;
- output size and BPW;
- tensor mix;
- MTP status;
- completion smoke output.

For KLD-based quality decisions, use the intended full KLD evidence for the
recipe and clearly record provenance. Keep p99 and p999 gates visible in quality
mode reports.


## Evidence Standard

A useful model report should keep:

- source GGUF and locked recipe;
- calibration corpus and imatrix;
- evaluation corpus and KLD base;
- code commit and CUDA device;
- PPL, mean KLD, p95/p99/p999 KLD, KLD tail mean, max KLD, RMS probability
  delta, same-top rate, top-flip weight, entropy drift, and non-finite counts;
- output size, BPW, tensor mix, MTP status, and scale tensor counts;
- at least one `llama-completion` smoke test.

Do not compare models using a proxy score, one first chunk, a tiny diagnostic
KLD sample, or mismatched calibration/evaluation provenance.


## Mixed NVFP4/MXFP6 Policy

Keep fused tensor decisions coherent. Treat related tensors as grouped decisions
where applicable:

- q/k/v attention projections;
- gate/up feed-forward projections;
- expert pairs and MoE-sensitive groups;
- MTP/NextN heads;
- output and token-embedding policy.

Use the documented mixed policies rather than scattered tensor-name exceptions.
Main-path fallback candidates such as `MXFP6_E2M3`, `Q4_K`, `Q6_K`, `Q8_0`,
`BF16`, and `F16` are valid when recipe policy, speed-aware candidate search,
or repair/edit workflows require them.

## Performance And Resource Discipline

- Never run overlapping GPU benchmarks.
- Serialize benchmark comparisons on a quiet GPU.
- Leave CPU headroom during large selector/KLD runs.
- Prefer checkpoints and resume over restarting broad searches.
- Be careful with KLD bases and logits files: they can be very large and become
  storage-bound.
- Do not change shared llama.cpp defaults or common parser behavior just to work
  around one local command line; first prove the issue with a minimal repro.

## Code Style

- Use existing llama.cpp/local helper APIs.
- Keep comments short and useful.
- Prefer structured parsers and data models over ad hoc string manipulation.
- Keep public vocabulary simple:
  - candidate search
  - best-candidate report
  - quality repair
  - four-over-six
  - mixed NVFP4/MXFP6
  - reuse/edit base
- Avoid publishing local machine paths, private run history, or internal cleanup
  notes in docs.

## Git Hygiene

- Check `git status --short` before editing.
- Do not revert changes you did not make unless the maintainer explicitly asks.
- Keep unrelated cleanup out of focused fixes.
- Commit meaningful completed checkpoints with clear messages.
- Before pushing or opening a release PR, scan public docs for private paths,
  stale names, internal cleanup notes, and local-only instructions.

## Maintainer Intent

This is an open-source, llama.cpp-based project maintained as a standalone
advanced GGUF quantizer. Keep it easy for users to build, run, inspect, repair,
and improve NVFP4/MXFP6 GGUF models without carrying unrelated llama.cpp app
surface.

---
> Source: [michaelw9999/advanced-gguf-quantizer](https://github.com/michaelw9999/advanced-gguf-quantizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
