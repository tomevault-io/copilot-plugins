## turboquant

> TurboQuant is a Rust library for research-grade vector quantization on LLM KV-cache style vectors. It provides MSE-oriented quantization, inner-product-oriented quantization, batch packing, quantized KV-cache helpers, a tract-based end-to-end ONNX decoder evaluation path, and an experimental Burn/WGPU batch backend.

# TurboQuant Agent Notes

## Project Identity

TurboQuant is a Rust library for research-grade vector quantization on LLM KV-cache style vectors. It provides MSE-oriented quantization, inner-product-oriented quantization, batch packing, quantized KV-cache helpers, a tract-based end-to-end ONNX decoder evaluation path, and an experimental Burn/WGPU batch backend.

Current status as of 2026-03-25: alpha. Suitable for local research, benchmarking, and library integration experiments. Not yet suitable as a drop-in production inference backend without additional system-specific validation.

## Architecture Map

- `src/turboquant_mse.rs`: stage-1 TurboQuant quantizer optimized for reconstruction error.
- `src/turboquant_prod.rs`: two-stage quantizer optimized for inner-product estimation.
- `src/qjl.rs`: standalone QJL residual sketch and estimators.
- `src/batch.rs`: packed batch containers and batch scoring/dequantization helpers.
- `src/kv_cache.rs`: single-head and multi-head quantized KV-cache wrappers.
- `src/real_model.rs`: tract/tokenizer-based ONNX decoder runner for end-to-end real-model evaluation.
- `src/backend.rs`: scalar/SIMD backend selection and low-level math kernels.
- `src/gpu.rs`: optional Burn/WGPU batch acceleration. MSE is fully accelerated; `TurboQuantProd` is hybrid GPU+CPU.
- `src/trace.rs`: safetensors trace loader for model-derived benchmark inputs.
- `src/codebook.rs`, `src/scalar_quant.rs`, `src/rotation.rs`, `src/bitpack.rs`, `src/utils.rs`: quantization primitives and shared utilities.
- `examples/benchmark.rs`: evaluation CLI for `synthetic`, `trace`, and `real-model` workloads.
- `scripts/export_hf_kv.py`: Hugging Face exporter for pre-RoPE Q/K/V traces.
- `scripts/export_hf_decoder_onnx.py`: Optimum-based ONNX export helper for lightweight decoder-only models.

## Tech Stack

- Rust `1.87.0`
- `nalgebra` for dense linear algebra
- `serde` / `serde_json` for serialization
- `thiserror` for typed errors
- `pulp` for runtime-dispatched SIMD
- `safetensors` for benchmark trace exchange
- `tract-onnx` for real-model ONNX decoder execution on CPU
- `tokenizers` for local tokenizer loading
- `burn` with `wgpu` feature for the optional GPU batch path
- `clap` for the benchmark CLI
- `criterion` for kernel benchmarks
- Optional Python tooling: `torch`, `transformers`, `optimum`, `onnx`, `numpy`, `safetensors`

## Commands

These commands should work from the repository root:

- `cargo fmt -- --check`
- `cargo clippy --all-targets --all-features -- -D warnings`
- `cargo test --all-features`
- `cargo check --examples --all-features`
- `cargo llvm-cov --workspace --all-features --summary-only`
- `cargo audit`
- `cargo run --release --example benchmark -- --workload synthetic --quick`
- `cargo run --release --example benchmark -- --workload real-model --real-model-dir artifacts/smollm2-135m-onnx --prompt "KV cache in one sentence." --real-eval-mode compare --bits 4 --real-key-strategy prod --max-new-tokens 16`
- `cargo run --release --example benchmark --features gpu -- --workload synthetic --quick --backend wgpu`

## Conventions

- Public boundaries should return typed `TurboQuantError` values for runtime failures. Do not silently clamp, truncate, or coerce malformed user input.
- Most core APIs assume finite, unit-norm vectors. Preserve that contract unless you are intentionally redesigning the API and updating tests/docs accordingly.
- The real-model path handles raw K/V tensors by storing per-vector norms separately before quantization. Do not use that as a reason to weaken the public quantizer contract.
- If you fix a bug, add a regression test in the same pass.
- If you change quantitative behavior, update the benchmark workflow or docs so the effect is measurable.
- `PolarQuant`, the WGPU backend, and the tract real-model runner are experimental surfaces. Keep that status explicit unless the guarantees materially improve.

## Critical Constraints

- Do not weaken validation to “accept more” malformed data unless the semantics are clearly defined and documented.
- Do not silently reinterpret invalid trace `query_positions`; invalid traces should fail loudly.
- Do not assume serialized `BatchQuantizedMSE` / `BatchQuantizedProd` payloads are trustworthy. Validate layout before decoding/scoring if they come from outside this process.
- Do not relabel synthetic or trace workloads as real-model runs.
- Do not claim the crate is production-ready for inference serving. It is still an alpha research library.

## Gotchas

- `ExecutionBackend::Simd` is the default CPU path; the crate still runs on platforms without wide SIMD because `pulp` dispatches at runtime.
- `TurboQuantProd::new` requires `bit_width >= 2`.
- `BatchQuantizedProd` stores raw packed sign bytes; layout validation is required to avoid indexing panics on malformed payloads.
- `KvTrace` expects model-derived tensors in pre-RoPE space and rejects negative or out-of-range query positions.
- The real-model tract path expects float32 decoder-only ONNX bundles with explicit past/present key/value tensors.
- Quantized real-model evaluation reconstructs float `past_key_values` tensors before feeding the next tract decode step.
- `src/main.rs` is a demo binary, not a stable CLI contract.

## Current State

- Implemented: scalar + SIMD CPU kernels, experimental WGPU batch acceleration, batch/KV-cache helpers, synthetic/trace benchmark paths, tract-based end-to-end ONNX decoder evaluation, Hugging Face trace exporter, ONNX export helper, CI-safe tiny ONNX fixture tests, CI lint/test/coverage/audit jobs.
- In progress: broader real-model coverage, clearer export compatibility notes, and more manual benchmark coverage on actual lightweight bundles.
- Known weak points: optional GPU dependencies still pull in a large experimental stack via Burn/WGPU; the tract path is CPU-oriented and currently reconstructs float caches between decode steps; the crate has no service-level observability because it is a library, not a daemon.

---
> Source: [AbdelStark/turboquant](https://github.com/AbdelStark/turboquant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
