## nightrun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

NightRun is a bare-metal x86_64 LLM appliance: a single `no_std` Rust UEFI
application that boots from USB and runs Llama 3.2 1B (Q8_0),
Qwen3-4B-Instruct-2507 (Q4_K_M) or Granite 4.1 3B (Q4_K_M, dense
transformer only — hybrid Granite is rejected at conversion) with no OS.
It deliberately **stays in UEFI Boot Services** (for USB keyboard, disk
reads, and MP services) — do not add `ExitBootServices`, and never call
firmware services from AP worker code (`nr-tensor::parallel` workers are
atomics + compute only).

## Commands

```sh
cargo test                    # host test suite (default-members exclude nr-boot)
cargo test -p nr-token        # single crate; -- <name> for a single test
cargo xtask build             # build BOOTX64.EFI
cargo xtask image [--model models/qwen3-4b-q4km.nrm]  # build nightrun.img
cargo xtask run [--img] [--model <file.nrm>] [--window] [--mem 4G|6G] \
    [--smp 8] [--secs N] [--shot t:file.png] [--keys "t:text\n"]
    # QEMU + OVMF; --keys supports <up>/<down>/<pgup>/<pgdn>/<esc> tokens;
    # Qwen runs need --mem 6G
cargo xtask bench             # scripted QEMU run -> docs/benchmarks.md
cargo run --release -p nrconvert -- in.gguf models/model.nrm
cargo run --release -p nrhost -- models/model.nrm --prompt "..." [--raw] \
    [--temp 0] [--threads 8]  # same engine on the host (debugging)
```

- `nr-boot` builds **only** via xtask: it needs nightly + `-Zbuild-std` and
  the custom hard-float target `x86_64-nightrun-uefi.json` (the builtin
  UEFI target is soft-float and breaks both AVX intrinsics and f32 perf).
  Never `cargo build --workspace` / `cargo test --workspace` — nr-boot's
  panic handler collides with std on the host target.
- Tests that need models (`nr-token` fixtures, `nr-model` parity) skip
  silently when `models/model.nrm` / `models/qwen3-4b-q4km.nrm` are
  absent; regenerate with nrconvert before trusting a green run. Parity
  tests serialize on an internal mutex (concurrent multi-GB loads OOM).
- QEMU testing of the model path needs `--img --mem 4G` (the model exceeds
  QEMU's virtual-FAT limit, so the ESP-directory dev mode boots without it).

## Architecture (read docs/architecture.md for the full picture)

- Crate layering: `nr-gfx`/`nr-ui` (framebuffer UI) and `nr-tensor`/
  `nr-token`/`nr-model` (engine) are platform-independent; `nr-boot` is the
  only crate that touches `uefi`. Engine crates are dual-target: `no_std`
  for the EFI build, `std` feature (default) for host tests and tools —
  keep new code `no_std`-clean (`libm` for float math, no `std::`).
- `tools/nrhost` runs the *identical* inference code path on Linux; debug
  engine issues there before reaching for QEMU. Serial (COM1) is the debug
  channel in QEMU: `target/serial.log`.
- `.nrm` format lives in `nr-model::format` (parser) and `tools/nrconvert`
  (writer); the tokenizer blob in `nr-token::blob` (parser) and
  `tools/nrconvert/src/tokenizer.rs` (writer). Change them in pairs and
  bump `VERSION`; `nr-token/tests/fixtures.rs` hardcodes header offsets.
- Inference correctness is pinned by greedy token-for-token parity with
  llama.cpp for BOTH families (`crates/nr-model/tests/parity.rs`). If you
  touch kernels, rope, or the forward pass and parity breaks, the code is
  wrong — not the fixture. Family gotchas that already bit once: Qwen3
  uses NEOX rope (half-split pairs) vs llama/granite's adjacent pairs,
  Qwen has no BOS, per-head Q/K RMSNorm before rope, and attention width
  (4096) != hidden width (2560). Granite has four muP scalars in the
  header (embed/attn/residual/logit; neutral values elsewhere — attn 0.0
  means 1/sqrt(head_dim)) and its /10 logit scaling makes greedy near-tie
  flips vs llama.cpp more common (top-2 gap < ~0.4): pin parity on
  prompts that match exactly, measure gaps with nrhost --debug-gap before
  suspecting the engine. Tokenizer fixtures (`tests/fixtures/tokenizer_cases*.txt`)
  come from `scripts/gen_tokenizer_fixtures.py` + official HF tokenizers
  (with split_special_tokens=True — user text never encodes to control
  tokens); the Qwen chat template is pinned to `apply_chat_template`.
- Prompt processing goes through `InferCtx::prefill_chunk` (batched, max
  64 tokens/pass, bit-identical to sequential `forward` — pinned by
  tests/prefill.rs); decode stays token-at-a-time. Model checksums are
  verified *while* loading (`nr-model::verify::StreamingVerifier`) — there
  is no post-load CRC pass. Chat commands: /clear (reset conversation,
  keep model), /bye (UEFI shutdown).
- No allocations in the generation loop: model tensors are zero-copy views
  into the loaded blob; KV cache + scratch come from the boot-time arena,
  sized from `InferCtx::required_bytes` — update it when adding buffers.
  Weight matrices are dtype-tagged (`QMat`: Q8_0/Q4_K/Q6_K); dispatch once
  per matvec, never inside row loops. After the model loads, storage is
  sealed (`modelload::seal_storage`) — any later disk read is a hard fault.
- Visual identity is centralized in `nr-gfx::theme`; screens must keep the
  synthwave look (verify with `--shot` screendumps, not by eye-balling
  code). The UI never claims fake capability: labels reflect what the
  build actually does.

---
> Source: [hardrave/NIGHTRUN](https://github.com/hardrave/NIGHTRUN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
