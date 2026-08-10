## moss-tts-cpp

> Durable reference for humans and agents maintaining this repo. Concise on

# Maintainer's guide — moss-tts.cpp

Durable reference for humans and agents maintaining this repo. Concise on
*why*; the README covers what users see.

## AI-assisted contributions

This project follows the same policy as the sibling repos (parakeet.cpp,
vibevoice.cpp) — the Linux kernel project's
[guidelines for AI coding assistants](https://docs.kernel.org/process/coding-assistants.html).
Commit rules:

- **No `Signed-off-by` from AI.** Only a human submitter signs off the DCO.
- **No `Co-Authored-By: <AI>` trailers.** The human contributor owns the change.
- **Use an `Assisted-by:` trailer** to attribute AI involvement. Format:
  `Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL]`
  (e.g. `Assisted-by: Claude:claude-opus-4-8 [Claude Code]`).
- The human submitter reviews, tests, and understands every generated line.

## What this project is

moss-tts.cpp is a C++17/ggml inference port of the OpenMOSS
**MOSS-Audio-Tokenizer** (the "Cat" — Causal Audio Tokenizer with Transformer),
a CNN-free, pure-Transformer RVQ neural codec: 24 kHz waveform ↔ 32 RVQ code
streams at 12.5 Hz. It runs entirely on stock ggml (no fork), with no
Python/ONNX/torch at inference time, and ships a flat C-API
(`include/moss_tts_capi.h`) for `dlopen`/FFI/LocalAI integration.

This is the **Foundation** (F1 repo scaffold + F2 audio tokenizer) of a larger
program: porting the whole MOSS-TTS family to ggml. The decomposition, ordered
dependency-first, is `F1 → F2 → V1 → V2 → V3 → V4`:

```
FOUNDATION (this repo)  shared by every variant
  F1  repo scaffold (CMake + ggml + dr_wav + loader + audio_io + C-API/CLI)
  F2  MOSS-Audio-Tokenizer: encoder (wav→codes) + decoder (codes→wav)
        every variant decodes its audio through F2
V1  MossTTSDelay (8B)      Qwen3 backbone + 33 emb/heads + delay SM + sampling
V2  MossTTSLocal (1.7B)    depth-transformer backbone, time-sync RVQ blocks
V3  MossTTSRealtime (1.7B) hierarchical text-audio inputs + streaming
V4  MOSS-TTS-Nano (~100M)  separate upstream repo, 48 kHz
```

F2 comes first because every variant's audio is produced by this codec, it is
the riskiest novel ggml work (ONNX-only upstream), and it can be validated fully
standalone (reconstruction round-trip vs the ONNX tokenizer — no backbone). The
full design + rationale is in
[`docs/superpowers/specs/2026-06-03-foundation-audio-tokenizer-design.md`](docs/superpowers/specs/2026-06-03-foundation-audio-tokenizer-design.md).

### Reference sources (single sources of truth, in order of trust)

1. **`OpenMOSS-Team/MOSS-Audio-Tokenizer`** (HF) — `config.json` (dims) +
   `modeling_moss_audio_tokenizer.py` (module logic). Authoritative.
2. **`Blaizzy/mlx-audio`** `mlx_audio/codec/models/moss_audio_tokenizer/` —
   clean non-PyTorch reference for the algorithms.
3. **`OpenMOSS-Team/MOSS-Audio-Tokenizer-ONNX`** — the encoder/decoder ONNX we
   benchmark + parity-test against (the upstream torch-free pipeline).
4. The model's `model.safetensors.index.json` — authoritative tensor names.

When a numeric bug is suspected, dump the matching tensor from (1) and diff.
Magnitude alone is not a reliable signal (the vibevoice lesson).

## Layout

```
include/
  moss_tts.h            C++ API (moss::Codec: load/sample_rate/num_quantizers/encode/decode/reconstruct)
  moss_tts_capi.h       flat C-API for FFI / dlopen
src/
  moss_tts.cpp          C++ API impl (version + thin Codec pimpl over AudioTokenizer)
  moss_tts_capi.cpp     flat C-API shim
  audio_tokenizer.{hpp,cpp}  orchestrator: wires encoder towers + quantizer + decoder towers
  model_loader.{hpp,cpp}     GGUF -> config + name->ggml_tensor map; builds towers from metadata
  backend.{hpp,cpp}     ggml backend init + persistent ggml_gallocr; compute_graph_with_inputs()
  patchify.{hpp,cpp}    CNN-free down/up reshape-permute subgraph (PatchedPretransform)
  transformer.{hpp,cpp} ProjectedTransformer: RoPE MHA + LayerScale + erf-GELU FFN + sliding-window mask
  quantizer.{hpp,cpp}   ResidualLFQ: encode (argmin loop) + decode (gather+sum)
  rope.hpp              RoPE cos/sin (interleaved-pair convention)
  audio_io.{hpp,cpp}    dr_wav load/save + linear resample to 24 kHz
  common.{hpp,cpp}      logging
  ggml_extend.hpp       small ggml helpers
examples/cli/main.cpp   moss-tts-cli: info | encode | decode | reconstruct
scripts/
  convert_audio_tokenizer_to_gguf.py  safetensors -> GGUF (WNConv fusion, identity-proj drop, stage metadata)
  gen_test_fixtures.py                numpy/scipy reference dumps for the model-independent parity tests
  gen_onnx_reference.py               upstream ONNX encoder+decoder -> ref_recon.wav + ref_codes.npy (parity ref)
  requirements.txt                    converter deps + reference-only (onnxruntime, soundfile)
tests/
  test_*.cpp            ctest targets; SKIP_RETURN_CODE=77 means "skipped"
  fixtures/             committed tiny GGUF + numpy reference dumps
bench.sh                reconstruction RTF: moss-tts-cli vs ONNX on samples/*.wav
docs/
  conversion.md         authoritative GGUF tensor-naming + converter reference
  superpowers/specs/    the F1+F2 design spec
third_party/ggml        pinned submodule (no fork)
models/                 output dir for converted GGUFs (gitignored)
```

### Memory model (carried from parakeet.cpp — do not regress)

- Subgraphs are built in a **`no_alloc=true`** `ggml_context`: tensor metadata
  exists but backing buffers do not until allocation. Inputs are then materialized
  via `backend::compute_graph_with_inputs` (`src/backend.hpp`), which does
  `ggml_gallocr_alloc_graph` → write each input's data → `compute`.
- Keep the **persistent `ggml_gallocr`** in `src/backend.cpp` reused across the
  many per-call graphs (no per-call alloc/free). Do NOT swap in
  `ggml_backend_sched` on the fast path — it re-plans the split every call and
  regressed CUDA in parakeet.cpp. Sched is a per-graph fallback only, used when
  the active GPU backend lacks an op kernel.
- **Zero-copy weights**: clone loader tensors by reference; never copy weights
  per call.
- Compute-pool sizes scale with sequence/sample length (the vibevoice lesson —
  hardcoded sizes silently pass on short clips and abort on long ones).

### Metadata-driven towers

The encoder and decoder are 8-module sequences alternating `PatchedPretransform`
(no weights) and `Transformer` stages. The loader builds the towers from the
explicit `moss.at.{enc,dec}.*` int32 stage arrays (`n_stages` + the parallel
per-stage arrays `kind`, `patch`, `in_dim`, `d_model`, `n_heads`, `n_layers`,
`d_ff`, `out_dim`, read via `get_i32_array`) — no source changes per stage. The
`moss.at.encoder_kwargs` / `decoder_kwargs` JSON strings are emitted for
provenance only; the loader does **not** parse them. Encoder patch ratios
`240,2,2,2` (product = 1920 = downsample); decoder mirrors `2,2,2,240`.

## Build

```
cmake -B build -DMOSS_TTS_BUILD_TESTS=ON -DMOSS_TTS_BUILD_EXAMPLES=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
ctest --test-dir build --output-on-failure
```

### CMake options

| Option                    | Default | Purpose                              |
| ------------------------- | ------- | ------------------------------------ |
| `MOSS_TTS_BUILD_TESTS`    | OFF     | Compile + register ctest targets     |
| `MOSS_TTS_BUILD_EXAMPLES` | OFF     | Build `moss-tts-cli`                 |
| `MOSS_TTS_BUILD_SERVER`   | OFF     | Build the server (placeholder)       |
| `MOSS_TTS_SHARED`         | OFF     | Build libmoss-tts as a shared lib    |
| `MOSS_TTS_TEST_LARGE`     | OFF     | Run large/E2E tests                  |
| `MOSS_TTS_GGML_CUDA`      | OFF     | Forward GGML_CUDA to the submodule   |
| `MOSS_TTS_GGML_METAL`     | OFF     | Forward GGML_METAL to the submodule  |
| `MOSS_TTS_GGML_VULKAN`    | OFF     | Forward GGML_VULKAN to the submodule |
| `MOSS_TTS_GGML_HIPBLAS`   | OFF     | Forward GGML_HIPBLAS to the submodule|

Runtime backend selection: `MOSS_TTS_BACKEND=cpu|cuda|...` (honored verbatim by
`src/backend.cpp` when the named backend is registered, else falls back).

### Profiling (per-stage wall-clock)

`src/profiler.{hpp,cpp}` provides `MOSS_PROFILE("stage")` RAII scopes that
accumulate per-stage time into an env-gated registry. **Off by default, zero
overhead, inference byte-identical** (the scope is a single bool check when off
and never touches inference tensors). Two ways to read it:

- `MOSS_TTS_PROFILE=1 ./build/bin/moss-tts-cli tts …` — prints the per-stage table
  to stderr at the end of any `tts`/`tts-*` run.
- `./build/bin/moss-tts-cli bench <delay|local|rt|nano> --model … --codec …
  --tokenizer … --text "…" [--warmup 1] [--iters 3] [--seed 12345]` — runs
  `--warmup` discarded then `--iters` measured generations and prints the table +
  averaged wall/audio + RTF.

Stages: `embed`, `backbone.prefill`, `backbone.decode`, `depth` (+ nested
`depth.head` for the per-channel local heads), `heads` (V1 top-level head),
`sample`, `codec`. The report separates one-time `backbone.prefill` from per-step
`backbone.decode` via the calls/avg columns. The threshold for the head-dot
`parallel_for` is independent (`MOSS_HEAD_PARALLEL_MIN`).

**CAVEAT:** tiny-fixture numbers are NOT representative (tiny hidden/vocab) — they
exercise the instrumentation for CI/regression only; real per-stage attribution
needs a real checkpoint. Complements (does not replace) the `bench_*.sh`
end-to-end RTF scripts.

## Tests at a glance

`moss_add_test()` sets `SKIP_RETURN_CODE=77`, so a test that returns 77 shows as
ctest SKIP. The model-independent tests run anywhere (no checkpoint); the
real-model tests SKIP until their env vars point at the 7GB checkpoint (and, for
parity, an ONNX reference).

| File                            | What it does                                              | Needs models? |
| ------------------------------- | -------------------------------------------------------- | :-----------: |
| `test_smoke`                    | Lib loads, version string non-empty                       | no            |
| `test_audio_io`                 | dr_wav load/save round-trip + resample                    | no            |
| `test_model_loader`             | GGUF -> config + name->tensor on a tiny fixture           | no            |
| `test_patchify`                 | down∘up == identity; fixed-pattern interleave check       | no            |
| `test_layernorm_gelu`           | LayerNorm (with bias) + exact erf-GELU vs numpy           | no            |
| `test_rope`                     | RoPE cos/sin tables vs reference                          | no            |
| `test_transformer_block`        | ProjectedTransformer forward numerics, tiny weights       | no            |
| `test_transformer_masked`       | Masked (sliding-window) transformer forward               | no            |
| `test_window_mask`              | Sliding-window causal mask construction                   | no            |
| `test_quantizer`                | ResidualLFQ argmin select + decode gather vs reference    | no            |
| `test_audio_tokenizer_e2e`      | Full encode→decode round-trip on tiny random weights      | no            |
| `test_audio_tokenizer_shapes`   | Real-model encode/decode shape gate                       | yes (env)     |
| `test_load`                     | Real GGUF opens; sr=24000, nq=32                          | yes (env)     |
| `test_reconstruct_parity`       | **Milestone gate**: our recon vs ONNX recon, SNR ≥ thresh | yes (env+ONNX)|

Env vars for the real-model tests (set whichever you need):

```
MOSS_TTS_TOKENIZER    # .gguf path (used by all three real-model tests)
MOSS_TTS_PARITY_IN    # input wav fed to the ONNX reference
MOSS_TTS_PARITY_REF   # OUT/ref_recon.wav from scripts/gen_onnx_reference.py
MOSS_TTS_BACKEND      # cpu|cuda|... (runtime backend; also used by bench.sh)
```

### Parity-fixture methodology

Each ggml op is pinned by a numpy/scipy reference: `scripts/gen_test_fixtures.py`
computes the expected tensors (patchify interleave, LayerNorm+GELU, RoPE tables,
transformer-block forward, quantizer argmin/gather) and dumps them to
`tests/fixtures/`; the matching `test_*.cpp` builds the same ggml subgraph on the
same tiny weights and asserts close agreement. This is how every op was validated
before the real checkpoint existed. Add a fixture when you add an op.

## GGUF tensor naming + the converter

`scripts/convert_audio_tokenizer_to_gguf.py` turns the upstream safetensors into
a single f32 GGUF. The full schema is authoritative in
[`docs/conversion.md`](docs/conversion.md); the load-bearing rules:

- **WNConv fusion**: encoder/decoder block projections and all quantizer
  projections are weight-normalized 1×1 convs (`weight_norm(dim=0)`). The
  converter fuses `g`/`v` (`parametrizations.weight.original0/1`) into a dense
  `(out,in)` `<name>.weight` and keeps the stored bias. Bit-exact vs torch.
- **Identity-projection drop**: block projections are present only when dims
  differ; when `in == d_model == out` the projection is absent and the C++ loader
  treats a missing `input_proj`/`output_proj` as identity. Do not synthesize it.
- **Stage-metadata arrays**: per-stage block configs + patch ratios are emitted
  as GGUF KV int32 arrays (`moss.at.{enc,dec}.{n_stages,kind,patch,in_dim,d_model,n_heads,n_layers,d_ff,out_dim}`)
  and the loader builds the towers from these. The `moss.at.encoder_kwargs` /
  `decoder_kwargs` JSON strings are also emitted but for provenance only — the
  loader does not parse them.
- Tensor names match the upstream PyTorch hierarchy **verbatim** (no `model.`
  prefix). Against the real checkpoint the converter reports 0 unmapped keys.

Always run the converter with `--strict` to fail on unmapped source keys.

## Gotchas / past-bug archaeology

From the spec's gotchas list + what the port confirmed. These will bite again.

1. **No input normalization.** Feed the raw float waveform in [-1,1]; only
   zero-pad to a multiple of 1920. No DC removal, no gain. (vibevoice normalized
   and broke things.)
2. **erf-GELU, not tanh.** Exact erf-GELU; FFN is two bias-free linears, NOT
   SwiGLU.
3. **LayerNorm has weight AND bias** (eps 1e-5). It is not RMSNorm.
4. **LayerScale** is a learnable per-channel vector multiplying the residual
   *branch* output, between the sublayer and the residual add. Never fold it into
   a linear.
5. **RoPE is interleaved-pair** (ggml `GGML_ROPE_TYPE_NORMAL`), head_dim 64,
   max_period 10000, applied to Q and K. Offline streaming offset = 0.
6. **MHA, not GQA.** Fused QKV in `self_attn.in_projs.0` `(3*d_model, d_model)`,
   bias-free; out_proj bias-free; SDPA scale `head_dim^-0.5`.
7. **Sliding-window causal mask is stage-dependent**: key j visible to query i
   iff `0 ≤ i-j < context`, `context = round(frame_rate_at_stage_input × 10s)`.
   Track `current_frame_rate` exactly through the module list (start 24000,
   divide per encoder downsample / multiply per decoder upsample after appending).
   For short clips this equals plain causal; implement the window so long audio
   matches the reference bit-for-bit.
8. **Patchify interleave order must mirror exactly on decode.** Down does
   `(D,T)→(D,T/p,p)→permute→(D*p,T/p)`; up mirrors. Codes layout is
   **ne0 = NQ, ne1 = T** (frame-major). Getting the interleave wrong silently
   corrupts audio.
9. **Quantizer L2-normalizes both** the projected encoding and the codebook
   (eps 1e-12) before the argmin in 8-dim codebook space; decode is argmin +
   gather + sum only — no EMA, straight-through, or extra scaling.
10. **Decoder output is the waveform directly** from the final patch-240
    unpatchify — no output conv, tanh, or clamp.
11. **gallocr input-set ordering**: with `no_alloc=true`, an input's buffer does
    not exist until `ggml_gallocr_alloc_graph`, so input data must be written
    *after* allocation, not at build time. Use
    `backend::compute_graph_with_inputs` (alloc → set inputs → compute); writing
    inline aborts.
12. **Bitrate control** = use the first `k` of 32 codebooks on encode/decode.

## Known follow-ups / not yet implemented

Genuine deferrals — recorded so they are not forgotten:

- **Bitrate-`k` control.** `encode`/`decode`/CLI/C-API currently always use all
  32 codebooks. The spec notes you can decode with the first `k` of 32
  (`k×0.125` kbps); exposing a `--n-quantizers k` control is a future
  enhancement. (`gen_onnx_reference.py` already accepts `--n-quantizers` on the
  ONNX side.)
- **Codes-exact-match assertion in the parity gate.**
  `gen_onnx_reference.py` emits `ref_codes.npy`, and the spec's parity DoD
  mentions asserting our encode codes match the ONNX codes above an exact-match
  rate, but `tests/test_reconstruct_parity.cpp` currently only checks
  reconstructed-waveform SNR. Wiring a codes comparison (and tuning the
  exact-match-rate threshold) is best done during the first real-model run.

## V1: MossTTSDelay (8B text-to-speech)

The first generative variant. Text (+ optional reference clip for voice cloning)
-> speech, decoded through the Foundation codec. Built dependency-first across
T1-T13; all 33 streams (1 text + 32 audio) are produced by one Qwen3 backbone
with 33 embedding tables and 33 output heads, sequenced by the "delay" staircase.

### Pipeline (one synthesis)

```
text + instruction/language + (reference wav)
  -> de.tokenizer (Qwen3 BPE)        : text -> token ids
  -> prompt builder                   : chat template + (codec.encode of the
                                        reference wav) spliced in on the delay
                                        staircase; produces the 33-row input grid
  -> DelayEmbeddings                  : per-row table lookup over 33 tables,
                                        SUMMED into one hidden vector per step
  -> Qwen3 backbone (+ KV cache)      : prefill the prompt, then autoregressive
                                        decode one step at a time
  -> 33 LM heads                      : text head + 32 audio heads -> 33 logit rows
  -> delay state machine              : decides, per step, which rows are active /
                                        padded (the staircase from delay_state.py)
  -> sampling                         : temp / top-k / top-p / rep-penalty,
                                        seedable (mt19937_64); or greedy
  -> (loop until audio_end / max_new_tokens)
  -> collected 32 audio-code streams  -> codec.decode -> 24 kHz waveform
  -> loudness_normalize                : RMS -> target -20 dBFS, gain clamped to
                                        [-3, +3] dB (fidelity match to upstream)
```

Voice cloning runs the reference wav through `codec.encode` and prepends those
codes (on the staircase) so the backbone continues the speaker.

### New `src/` files and their roles

```
qwen3.{hpp,cpp}            one Qwen3 decoder layer: per-head q/k RMSNorm, no
                           biases, GQA, RoPE, SwiGLU FFN, RMSNorm
delay_backbone.{hpp,cpp}   stack of Qwen3 layers + KV cache (prefill + decode)
delay_embeddings.{hpp,cpp} 33-table embedding lookup, summed to one hidden/step
lm_heads.{hpp,cpp}         text head + 32 audio heads (pad code masked out)
delay_state.{hpp,cpp}      delay staircase state machine (port of delay_state.py)
sampling.{hpp,cpp}         temp/top-k/top-p/rep-penalty sampler, seedable RNG
de_tokenizer.{hpp,cpp}     Qwen3 BPE tokenizer (pre-tokenizer regex approximated)
prompt.{hpp,cpp}           chat-template prompt builder + delay splice + cloning
delay_constants.hpp        token ids, table widths (1025), pad code (1024), etc.
moss_tts_delay.{hpp,cpp}   DelayTTS orchestrator: load + tts(); the pimpl behind
                           moss::Delay in include/moss_tts.h
```

`moss::Delay` (the public API) is a thin pimpl over `DelayTTS`. CLI: `moss-tts-cli
tts --model BACKBONE --codec CODEC --tokenizer TOK --text "..." [--reference R.wav]
[--seed N] [--greedy] --out OUT.wav`.

### Converters

- `scripts/convert_moss_tts_delay_to_gguf.py` -> one GGUF holding the Qwen3
  backbone (`qwen3.*` tensors + `qwen3.{hidden,n_layers,n_heads,n_kv_heads,
  head_dim,intermediate,rope_base,rms_eps,text_vocab}` metadata), the 33
  embedding tables + 33 heads (`de.embed_tokens`, `de.emb_ext.{0..31}`,
  `de.lm_head_text`, `de.lm_head_audio.{0..31}`), and `moss.de.*` metadata
  (n_vq, audio_vocab, sample_rate, special token ids). Run with `--strict`.
  Emits an f32 backbone (~30 GB for 8B); quantize it with the script below.
- `scripts/convert_tokenizer.py` -> the BPE tokenizer GGUF for `de_tokenizer`.
- `scripts/quantize_gguf.py` -> selective, allowlist-based GGUF quantizer
  (auto-detects `--kind delay` from `qwen3.*`/`de.*` tensors; also handles the
  Foundation `codec` GGUF). See **Backbone quantization** below.

### Backbone quantization (`scripts/quantize_gguf.py`)

Shrinks the 8B Delay backbone (~30 GB f32 -> ~5 GB) by re-typing ONLY the
Qwen3 matmul weights, which are the only backbone tensors read through
`ggml_mul_mat` (qwen3.cpp), and `ggml_mul_mat` dequantizes the `Q*_K`/`Q8_0`
row formats natively.

- **QUANTIZE** (to `--type`, default `q8_0`): the Qwen3 backbone matmuls only —
  `qwen3.blk.{i}.attn_{q,k,v,o}.weight`, `qwen3.blk.{i}.ffn_{gate,up,down}.weight`.
- **KEEP F32 (the #1 footgun — do NOT quantize):**
  - **ALL `de.*` tensors** (`de.embed_tokens`, `de.emb_ext.{0..31}`,
    `de.lm_head_text`, `de.lm_head_audio.{0..31}`). DelayEmbeddings (CPU table
    gather) and the LM heads (CPU dot products) read these as **raw float32**
    (host-staged at load via `read_tensor_f32`, then CPU gather / dot) — they are
    NOT routed through `ggml_mul_mat`, so quantizing them corrupts the reads. They
    must stay f32.
  - all norm/scale vectors: `qwen3.blk.{i}.{attn_norm,attn_q_norm,attn_k_norm,
    ffn_norm}.weight` and `qwen3.output_norm.weight` (small; full precision).
- All KV metadata is copied verbatim.

Note: pure-python K-quant *encoding* (q4_k/q5_k/q6_k) needs a recent gguf-py;
if unavailable the script errors cleanly — use `q8_0`/`q4_0` here, or run
llama.cpp's `llama-quantize` on the f32 GGUF (same allowlist applies).

### Tests

CI parity fixtures (model-independent, pinned by numpy or upstream-python; run
anywhere):

| Test                  | Pins                                                   |
| --------------------- | ------------------------------------------------------ |
| `test_qwen3_block`     | one Qwen3 layer forward (q/k RMSNorm, RoPE, SwiGLU)    |
| `test_delay_kv`        | KV-cache prefill+decode equals a full re-run           |
| `test_delay_embeddings`| 33-table lookup summed to one hidden vector            |
| `test_lm_heads`        | text + 32 audio heads, pad-code masking                |
| `test_sampling`        | temp/top-k/top-p/rep-penalty + seeded determinism      |
| `test_delay_state`     | the delay staircase vs the python reference            |
| `test_de_tokenizer`    | BPE token-id parity                                    |
| `test_prompt`          | chat template + delay splice + cloning layout          |

Env-gated real-model gates (SKIP/77 without their env vars):

| Test                | Needs                                                          |
| ------------------- | ------------------------------------------------------------- |
| `test_backbone_parity` | `MOSS_TTS_DELAY` + `MOSS_DE_REF_DUMP` (logit parity vs dump)|
| `test_e2e_tts`         | `MOSS_TTS_DELAY` + `MOSS_TTS_TOKENIZER` + `MOSS_DE_TOKENIZER`|
| `test_closed_loop`     | the e2e three + `MOSS_PARAKEET_CLI` + `MOSS_PARAKEET_MODEL`  |

`test_e2e_tts` runs the full stack and checks the wav is 24 kHz, > 0.3 s,
audible, and unclipped. `test_closed_loop` synthesizes "the quick brown fox..."
(seed 12345), shells out to a parakeet ASR CLI (verified contract: `<cli>
transcribe --model M --input W` -> transcript on stdout; the test is env-gated
so it SKIPs without the parakeet vars), then asserts >= 70% input-word recall.

### V1 gotchas (these will bite again)

1. **Qwen3 q/k per-head RMSNorm.** Each attention head's q and k are RMSNorm'd
   *individually* (head_dim-wide), before RoPE. Not a single norm over the whole
   projection. Biases are absent throughout the backbone.
2. **Do NOT normalize the summed input embedding.** DelayEmbeddings sums the 33
   per-row table lookups into the hidden vector and feeds it straight in — no
   RMSNorm/scaling on the sum (the analog of the Foundation "no input norm" bug).
3. **1025-wide audio tables / pad code 1024.** Each audio table/head is 1025
   wide: 1024 real codes + index 1024 = the pad/delay code. Heads mask the pad
   code out of sampling; embeddings have a real row for it (delayed/inactive
   rows embed the pad code).
4. **Delay staircase ported line-for-line from `delay_state.py`.** The per-step
   active/padded decision is a faithful port; when it drifts, diff against the
   python reference (`test_delay_state` pins it), don't "fix" it by intuition.
5. **Determinism via `--seed` / our `mt19937_64`.** Same seed -> same audio on
   the same build. Exact bit-match against the upstream torch RNG is explicitly
   out of scope (different PRNG); parity is checked at the logit level
   (`test_backbone_parity`), not the sampled-token level.
6. **Tokenizer pre-tokenizer regex is an approximation for non-ASCII.** The
   Qwen3 BPE pre-tokenizer split is approximated; ASCII English matches upstream,
   exotic Unicode pre-token boundaries may differ. `test_de_tokenizer` pins the cases
   we rely on.
7. **Loudness normalization is applied last.** `DelayTTS::tts` runs
   `loudness_normalize` on the assembled `*wav` just before returning (port of
   `pipeline.py`: RMS -> -20 dBFS target, gain clamped to [-3, +3] dB). It's a
   fidelity match to the upstream output level, not an intelligibility step;
   it scales the whole waveform by one factor (skipped if empty).

### Known follow-ups (V1)

- **GPU `->data` path for embeddings/heads — DONE.** The library is now
  device-safe: `moss::read_tensor_f32`/`read_tensor_i32` (in `backend.hpp`)
  host-stage the embed/head weight tables into owned `std::vector`s at load (the
  gather/dot math is unchanged → CPU byte-identical), and `local_adapters`
  `to_local`/`head_logits` use the device-safe graph-with-inputs pattern
  (`ggml_set_input`/`ggml_set_output` + `compute_graph_with_inputs` +
  `ggml_backend_tensor_get`), so `src/` is grep-clean of tensor `->data` and a
  GPU build (`-DMOSS_TTS_GGML_CUDA/METAL/VULKAN=ON`) is correct-by-construction.
  Residuals: (a) GPU execution is unverified on this CPU-only box — a real
  CUDA/Metal/Vulkan smoke-run is deferred to GPU hardware; (b) host-staging
  duplicates the f32 embed/head tables in host RAM — split-buffer placement (no
  duplication) is a future follow-up; (c) the category-B test-internal-ctx
  ggml-op unit tests (`test_rope`/`test_transformer_block`/`test_transformer_masked`/
  `test_patchify`/`test_layernorm_gelu`/`test_moss_tts_mlp`/`test_quantizer`/
  `test_partial_decode`/`test_window_mask`) still build their own CPU ctx +
  `compute_graph` and read computed outputs via `->data` (most also read
  loader-allocated input/reference fixtures via `->data` — the same loader-backed
  read class fixed elsewhere); they don't exercise the library's device-safety, and
  because the computed-output reads need the full set_input/set_output rework
  regardless, each is deferred as a whole-file conversion for a GPU `ctest`.
- **Host head dot products are multi-threaded — DONE (2026-06-05).** The LM/audio
  heads keep the deliberate host-staged f32 design (gathered/dotted on the host,
  not routed through `ggml_mul_mat`, so never quantized), but the per-step
  vocab-row loops in `lm_heads` / `rt_heads` / `nano_heads` now run through
  `moss::parallel_for` (`src/parallel.{hpp,cpp}`, std::thread chunked, threshold
  via env `MOSS_HEAD_PARALLEL_MIN`, default 512; the moss-tts target now links
  `Threads::Threads`). Each logit is an independent dot with its accumulation
  order untouched → byte-identical; the dominant text head (~311M scalar MACs/step
  for V1) now uses all cores instead of one. (The V2-local `head_logits` already
  runs through ggml `mul_mat`, so it was already threaded; the embedding gather is
  a cheap row memcpy, left serial.) Validated by `test_parallel_for` (parallel==
  serial + exact `[0,n)` coverage across edge cases) + the head parity tests at the
  same maxerr, including forced-parallel (`MOSS_HEAD_PARALLEL_MIN=1`) reruns on the
  tiny fixtures (`test_{lm,rt,nano}_heads_parallel`).
- **Audio rep-penalty at penalty != 1 is untested.** The repetition-penalty path
  exists in `sampling.cpp` but has only been exercised at penalty == 1.
- **Pre-tokenizer non-ASCII heuristic** (see gotcha 6) — revisit if a non-English
  prompt mis-tokenizes.

### Closing V1 on real hardware

```bash
# 1. Convert the 8B Delay backbone, the codec, and the tokenizer
.venv/bin/python scripts/convert_moss_tts_delay_to_gguf.py \
    --model OpenMOSS-Team/MOSS-TTS \
    --out models/moss-tts-delay-f32.gguf --strict
#    (codec = the Foundation MOSS-Audio-Tokenizer GGUF; tokenizer via convert_tokenizer.py)
.venv/bin/python scripts/convert_tokenizer.py --model OpenMOSS-Team/MOSS-TTS \
    --out models/moss-de-tokenizer.gguf

# 2. Produce the logit-parity reference dump
.venv/bin/python scripts/gen_delay_reference.py --model OpenMOSS-Team/MOSS-TTS \
    --out /tmp/ref_dump.gguf

# 3. Wire env vars + run the env-gated gates
export MOSS_TTS_DELAY=models/moss-tts-delay-f32.gguf
export MOSS_DE_REF_DUMP=/tmp/ref_dump.gguf
export MOSS_TTS_TOKENIZER=models/moss-audio-tokenizer-f32.gguf
export MOSS_DE_TOKENIZER=models/moss-de-tokenizer.gguf
ctest --test-dir build -R 'test_backbone_parity|test_e2e_tts' --output-on-failure

# 4. Closed-loop WER gate needs a parakeet.cpp build + ASR model
export MOSS_PARAKEET_CLI=/path/to/parakeet-cli
export MOSS_PARAKEET_MODEL=/path/to/parakeet-asr.gguf
ctest --test-dir build -R test_closed_loop --output-on-failure

# 5. Benchmark TTS tokens/sec + RTF
./bench_tts.sh cpu  --model models/moss-tts-delay-f32.gguf \
    --codec models/moss-audio-tokenizer-f32.gguf --tokenizer models/moss-de-tokenizer.gguf
```

## V2: MossTTSLocal (RQ-Transformer text-to-speech)

The second generative variant. Same text(+reference) -> speech contract as V1,
but the 33 channels (1 text + 32 audio) are produced by an **RQ-Transformer**
instead of the delay staircase: a **global** Qwen3 backbone runs over *time*,
and at every timestep a small **local (depth) transformer** runs over the 33
*channels* to emit all of them for that frame. There is **NO delay pattern** —
every frame is fully populated. Generation stops on `audio_end`.

### Pipeline (one synthesis)

```
text + instruction/language + (reference wav)
  -> de.tokenizer (Qwen3 BPE)        : text -> token ids
  -> prompt_local builder             : chat template + (codec.encode of the
                                        reference wav) spliced in WITHOUT any
                                        delay; produces the 33-row input grid
  -> LocalEmbeddings (global sum)     : per-channel emb_list lookup over 33
                                        tables, SUMMED into one hidden/step
  -> global Qwen3 backbone (+KV cache): RoPE on; prefill the prompt then
                                        autoregressive decode, one timestep/step,
                                        producing a global hidden vector h_t
  -> per timestep, the DEPTH LOOP (the time x depth keystone):
       seed the local seq with h_t, then for channel c = 0..32:
         LocalAdapters.to_local       : project running local hidden -> local dim
         local (depth) transformer    : 4 layers, NO RoPE, q/k-norm KEPT; per-frame
                                        KV cache (reset() once, then step() one
                                        channel at a time -> O(channels))
         LocalAdapters.head_logits    : per-channel out-MLP + head_norm + lm_head
         sample channel c (or greedy) : audio rep-penalty 1.1 default
         LocalEmbeddings (re-embed)   : embed the just-sampled code via the SAME
                                        emb_list table and append to the local seq
  -> 32 audio-code streams            -> codec.decode -> 24 kHz waveform
  -> loudness_normalize                : RMS -> -20 dBFS (same as V1)
```

Voice cloning is identical in spirit to V1 (codec.encode the reference, prepend
its codes), but on the flat grid — no staircase.

### New / modified `src/` files

```
qwen3.{hpp,cpp}            REUSED for BOTH transformers, gated by a `use_rope`
                           flag: global = RoPE on; local = RoPE OFF (q/k-norm
                           still applied). Same layer code, two configs.
local_transformer.{hpp,cpp}  the 4-layer no-RoPE depth transformer; per-frame KV
                           cache (reset()/step(), mirroring rt_local) -> O(channels)
local_embeddings.{hpp,cpp} the 33-table emb_list: global SUM (prompt/prefill)
                           AND single-code re-embed (inside the depth loop)
local_adapters.{hpp,cpp}   in-MLP (h_t -> local seed) + per-channel out-MLP /
                           head_norm / lm_head (to_local + head_logits)
moss_tts_mlp.hpp           the SwiGLU MossTTSMLP used by the adapters (two
                           DISTINCT-dim variants; no bias, no pre-norm)
prompt_local.{hpp,cpp}     chat-template prompt builder, NO delay splice
moss_tts_local.{hpp,cpp}   LocalTTS orchestrator: load + tts(); pimpl behind
                           moss::Local in include/moss_tts.h
```

The `qwen3_load_layer` tensor-prefix loader was generalized so the same loader
reads both `qwen3.blk.{i}.*` (global) and `local.blk.{i}.*` (depth) layers.
`moss::Local` (public API) is a thin pimpl over `LocalTTS`. CLI: `moss-tts-cli
tts-local --model LOCAL --codec CODEC --tokenizer TOK --text "..." [--reference
R.wav] [--seed N] [--greedy] --out OUT.wav`.

Note the local transformer has **head_dim (128) != local_hidden (1536)** and
**n_heads*head_dim != hidden** — these are derived from the real q/k/v_proj
tensor shapes, not assumed equal (see the converter note below).

### Converter

`scripts/convert_moss_tts_local_to_gguf.py` -> one GGUF with three families:

- `qwen3.*` — the **global** backbone (tensors + `qwen3.{hidden,n_layers,
  n_heads,n_kv_heads,head_dim,intermediate,rope_base,rms_eps,text_vocab}`).
- `local.*` — the **depth** transformer (`local.blk.{i}.*`, `local.output_norm`).
  Its `head_dim` / `n_heads` / `n_kv_heads` are **DERIVED FROM THE REAL local
  q/k_proj shapes** (q_proj rows / head_dim = n_heads; k_proj rows / head_dim =
  n_kv_heads; head_dim = q_norm length). For this checkpoint: head_dim=128,
  n_heads=16, n_kv_heads=8, with local_hidden=1536 (so head_dim != local_hidden).
- `lc.*` — embeddings/adapters/heads: `lc.embed.{0..32}.weight`,
  `lc.in_mlp.{gate,up,down}.weight`, `lc.out_mlp.{0..32}.{gate,up,down}.weight`,
  `lc.head_norm.{0..32}.weight`, `lc.lm_head.{0..32}.weight`.

Quant allowlist (`scripts/quantize_gguf.py`, auto-detects the `local` kind from
`qwen3.*` + `local.*` tensors): QUANTIZE only the matmul weights of **both**
transformers — `qwen3.blk.{i}.attn_{q,k,v,o}` / `ffn_{gate,up,down}` and
`local.blk.{i}.attn_{q,k,v,o}` / `ffn_{gate,up,down}`. KEEP F32: **all `lc.*`
tensors** (read as raw float32 — host-staged via `read_tensor_f32`, gathered/dotted
host-side — so quantizing corrupts them) and **all `*_norm.weight`** (q/k-norm, attn/ffn norm, both
`output_norm`s).

### Tests

CI parity fixtures (model-independent; run anywhere):

| Test                   | Pins                                                  |
| ---------------------- | ----------------------------------------------------- |
| `test_local_block`      | one no-RoPE local layer (q/k-norm kept, no RoPE)     |
| `test_local_transformer`| the 4-layer depth stack, per-row KV step() parity     |
| `test_moss_tts_mlp`     | the SwiGLU MossTTSMLP (both distinct-dim variants)   |
| `test_local_embeddings` | emb_list global-sum AND single-code re-embed         |
| `test_local_adapters`   | in-MLP to_local + per-channel out/head_norm/lm_head  |
| `test_depth_loop`       | the time x depth keystone: a tiny-complete frame      |
| `test_prompt_local`     | chat template, NO delay splice                       |

Env-gated real-model gates (SKIP/77 without their env vars):

| Test                    | Needs                                                       |
| ----------------------- | ----------------------------------------------------------- |
| `test_local_parity`      | `MOSS_TTS_LOCAL` + `MOSS_LOCAL_REF_DUMP` (global+local logit parity) |
| `test_e2e_local`         | `MOSS_TTS_LOCAL` + `MOSS_TTS_TOKENIZER` + `MOSS_DE_TOKENIZER` |
| `test_closed_loop_local` | the e2e three + `MOSS_PARAKEET_CLI` + `MOSS_PARAKEET_MODEL`  |

`test_e2e_local` runs the full V2 stack and checks the wav is 24 kHz, > 0.3 s,
audible, and unclipped. `test_closed_loop_local` synthesizes "the quick brown
fox..." (seed 12345), shells out to a parakeet ASR CLI (verified contract:
`<cli> transcribe --model M --input W` -> transcript on stdout; the test is
env-gated so it SKIPs without the parakeet vars), then asserts >= 70% input-word
recall.

### V2 gotchas (these will bite again)

1. **No-RoPE local attention, but q/k-norm KEPT.** The depth transformer turns
   RoPE OFF (`use_rope=false`) yet still applies per-head q/k RMSNorm. Don't drop
   the norms when you drop RoPE.
2. **Two distinct-dim SwiGLU MossTTSMLP adapters, NO bias / NO pre-norm.** The
   in-MLP and out-MLP have different in/out dims and carry neither a bias nor a
   pre-norm. The local transformer's `head_dim*n_heads != local_hidden`, so dims
   genuinely differ — do not "simplify" them to a single square MLP.
3. **The emb_list is SHARED for global-sum and local-reembed.** The same 33-table
   `lc.embed.*` does the prefill global SUM and the in-loop single-code re-embed.
   One table set, two call sites.
4. **The depth loop uses a per-frame local KV cache.** Each timestep `reset()`s
   the local transformer, then `step()`s one channel at a time (each new token
   attends to the cached K/V for channels `0..c`), so the loop is O(channels). Don't
   forget the per-timestep `reset()` before the channel loop.
5. **Audio rep-penalty 1.1 by default, seeded from per-channel prompt history.**
   Unlike V1 (penalty 1.0), the Local audio sampler defaults to 1.1 and seeds the
   penalty history from each channel's prompt codes.
6. **The `maybe_cont` seq_kv=1 fix.** The continuation/decode path must use
   seq_kv=1 for the single new timestep; a wrong seq_kv silently corrupts the
   global KV-cache attention. Pinned by `test_depth_loop`.
7. **`n_heads*head_dim != hidden` in the local.** Don't assume the projection
   output width equals the hidden width — it doesn't here.

### KNOWN FOLLOW-UPS (V2)

1. **`prompt_local.gguf` fixture provenance.** The `test_prompt_local` fixture
   was dumped from a **faithful NUMPY RE-DERIVATION** of the Local processor
   (torch was unavailable in the venv), **not** the genuine
   `MossTTSDelayProcessor`. Recommend a one-time torch-box cross-check against the
   real processor on the first real-model run, then re-bless the fixture if it
   drifts.
2. **Depth-loop performance** — DONE. The local depth transformer now uses a
   per-frame KV cache (`LocalTransformer::step`/`reset`, mirroring `rt_local`), so
   the depth loop is **O(channels)** instead of O(channels^2). And
   `LocalTransformer::step` / `LocalAdapters::to_local` / `head_logits` now reuse
   persistent scratch contexts (`make_ctx_buf`) instead of allocating a fresh ggml
   ctx per call. The change is numerically identical (validated by `test_depth_loop`
   exact codes + `test_local_transformer` per-row step parity).
   **Per-step ctx-reuse is now COMPLETE across ALL per-step builders** (2026-06-05):
   the four that still `make_ctx(256 MB)` every call — `DelayBackbone::run`,
   `NanoBackbone::run`, `RtLocal::step`, `NanoLocal::step` — now hold a
   `std::vector<uint8_t> scratch_` sized once at load to
   `ggml_tensor_overhead()*2*B + ggml_graph_overhead_custom(B,false) + 1 MB`
   (B = the `ggml_new_graph_custom` node budget: 4096, or 8192 for `NanoBackbone`)
   and build every step's graph in it via `make_ctx_buf`. Byte-identical (the
   per-builder keystones `test_delay_kv` / `test_nano_backbone` /
   `test_rt_depth_loop` / `test_nano_frame_loop` pass at the same numbers); removes
   the per-step mmap/munmap + page-fault churn (most impactful in the depth loops,
   called 16–32× per frame). Steady RSS +~3–7 MB per converted instance — the tight
   bound deliberately avoids the 64/256 MB alternatives so Nano stays light. (The
   codec's full encode/decode `make_graph_ctx` is one-shot per clip, not per-step,
   so it is intentionally left as-is; the codec STREAMING step already uses
   `make_ctx_buf`.)
3. **GPU `->data` path — DONE.** Embeddings / heads host-stage their weight tables
   via `moss::read_tensor_f32`/`read_tensor_i32` at load and `local_adapters` uses
   the device-safe graph-with-inputs pattern (CPU byte-identical; `src/` grep-clean
   of tensor `->data`); a GPU build is correct-by-construction. See the V1 follow-up
   for the full description + residuals (GPU-unverified here; host-RAM duplication;
   category-B ggml-op unit tests).

### Closing V2 on real hardware

```bash
# 1. Convert the Local backbone + the BPE tokenizer (codec = the Foundation's,
#    shared with V1; no separate codec conversion needed).
.venv/bin/python scripts/convert_moss_tts_local_to_gguf.py \
    --model OpenMOSS-Team/MOSS-TTS-Local-Transformer \
    --out models/moss-tts-local-f32.gguf --strict
.venv/bin/python scripts/convert_tokenizer.py --model OpenMOSS-Team/MOSS-TTS \
    --out models/moss-de-tokenizer.gguf

# 2. Produce the global+local logit-parity reference dump
.venv/bin/python scripts/gen_local_reference.py \
    --model OpenMOSS-Team/MOSS-TTS-Local-Transformer --out /tmp/local_ref.gguf

# 3. Wire env vars + run the env-gated gates
export MOSS_TTS_LOCAL=models/moss-tts-local-f32.gguf
export MOSS_LOCAL_REF_DUMP=/tmp/local_ref.gguf
export MOSS_TTS_TOKENIZER=models/moss-audio-tokenizer-f32.gguf
export MOSS_DE_TOKENIZER=models/moss-de-tokenizer.gguf
ctest --test-dir build -R 'test_local_parity|test_e2e_local' --output-on-failure

# 4. Closed-loop WER gate needs a parakeet.cpp build + ASR model
export MOSS_PARAKEET_CLI=/path/to/parakeet-cli
export MOSS_PARAKEET_MODEL=/path/to/parakeet-asr.gguf
ctest --test-dir build -R test_closed_loop_local --output-on-failure

# 5. Benchmark wall-time + RTF
./bench_local.sh cpu --model models/moss-tts-local-f32.gguf \
    --codec models/moss-audio-tokenizer-f32.gguf --tokenizer models/moss-de-tokenizer.gguf
```

### v1.5 support (MOSS-TTS-Local-Transformer-v1.5)

**Delay vs Local (the naming that ends the confusion).** The V2 section above
("MossTTSLocal") describes the engine's `local.arch=qwen3` path. That path is really
the **Delay** arm: `OpenMOSS-Team/MOSS-TTS-Local-Transformer` (`model_type:
moss_tts_delay`), n_vq=32, 24 kHz mono, local depth transformer = Qwen3 4-layer
(no-RoPE) + in/out MLP adapters + head_norm + 1025-wide composite heads. v1.5 is a
**distinct** arm with a materially different architecture; do not treat it as "the
same Local path with flags". See the design doc
`docs/superpowers/specs/2026-07-10-v1_5-real-gptj-port-design.md` for the full
ground-truth port (supersedes the earlier speculative v1.5 notes).

**Local v1.5** = `OpenMOSS-Team/MOSS-TTS-Local-Transformer-v1.5` (`model_type:
moss_tts_local`), n_vq=12, 48 kHz **stereo**. Real architecture (438 bf16 tensors +
`config.json`, offline-verified):

- **Backbone** = Qwen3 36-layer (hidden 2560) under the `transformer.*` prefix, and
  `transformer.embed_tokens` **is used** (text input embed + tied text head), not
  frozen/skipped. The converter maps it to `lc.embed.0.weight` (the text input
  embedding table); `text_lm_head` is SKIPPED (tied to it). It is NOT emitted as
  `qwen3.token_embd`.
- **Local depth transformer = GPT-J, not Qwen3** (`gpt2_config`: n_embd 2560,
  n_head 32 -> head_dim 80 MHA no-GQA, n_layer **1**, n_inner 9728). Per block:
  LayerNorm **with bias** (`ln_1`/`ln_2`, eps 1e-6, not RMSNorm), **fused** `c_attn`
  QKV, interleaved **RoPE** (base 1e6) on q/k, scaled-dot attention (scale
  1/sqrt(80)), 2-matrix **silu** MLP (`fc_in`/`fc_out`, not gated SwiGLU),
  `local_transformer.ln_f` as output norm. `wte = nn.Identity()` (fed hidden states
  directly). Emitted under `local.*` with metadata `local.arch="gptj"`,
  `local.norm_type="layernorm"`, `local.rope_base=1e6`, `local.ln_eps=1e-6`.
- **Bare, weight-tied heads (no adapters)**: `text_lm_head` tied to
  `transformer.embed_tokens`; `audio_lm_heads.{0..11}` tied to
  `audio_embeddings.{0..11}`; `local_text_lm_head[2,2560]` binary head. Applied
  **directly** to the local hidden (`lm_head[c] . local_out`): no in/out MLP, no
  head_norm. `_global_hidden_to_local` is the identity (both 2560). Metadata flag
  `lc.bare_heads=1` drives this C++ head path. There is NO
  `speech_embedding_to_local_mlp`, `local_to_speech_embedding_mlps`, or
  `layer_norm_before_lm_heads`.
- **Audio vocab 1024, not 1025** (`lc.audio_vocab=1024`): pad code 1024 is masked to
  zero in the embed-sum (`audio_embeds * valid_mask`), never embedded. No baked pad
  row. Audio channels still mask the pad code to -inf in the head logits.
- **Binary channel-0 decision head** (`lc.local_text_head_mode=1`): a direct 2-wide
  `lc.local_text_head.weight` -> argmax {continue = `gen_slot`, stop = `audio_end`}.
  Special-token ids come from the real config (`audio_start=151669`,
  `audio_end=151670`, `gen_slot=151656`, ...) read from GGUF metadata.
- **MOSS-Audio-Tokenizer-v2**, 48 kHz stereo, 32 codebooks **decoded at depth 12**
  (partial first-k decode; codec quantizer check is `>=`, a 32-codebook codec serves
  a 12-code model). `moss_tts_local` emits interleaved stereo; `Local::channels()`
  drives the CLI's `save_wav(..., n_channels)`. The v2 codec decode is now
  parity-gated (dequant + 12 decoder stages + stereo interleave) via
  `test_codec_v2_parity` (step 5c below); the C++ codec loader is name-tolerant
  (v2 `self_attn.in_proj` / `ffn.0` / `ffn.2` with a legacy Foundation
  `in_projs.0` / `linear1` / `linear2` fallback).

**Depth loop (v1.5).** 12 local positions (0..11): the local transformer runs once
on the pos-0 global hidden; the binary text decision (channel 0) and audio codebook
0 both read that pos-0 hidden; codebook c reads position c; only
`audio_embeddings[c]` feed the local transformer (no text/slot feed).
`to_local(global hidden)` is the identity for GPT-J.

**Coexistence.** The converter auto-detects the arm by config shape (`gpt2_config`
present -> v1.5 Local -> `local.arch="gptj"`; else Delay -> `local.arch="qwen3"`).
The C++ loader reads `local.arch` once and dispatches local-block construction +
forward. Absent (older GGUF) => `"qwen3"` => byte-identical Delay behavior. bf16
checkpoints load via a torch `framework="pt"` -> f32 fallback.

**Offline validation** (always-run, no real model): `test_local_v15_engine`
(op-level GPT-J block + bare-head + pad-mask gate on a tiny synthetic fixture from
`gen_test_fixtures.py`), plus the older schema fixtures `test_local_v15_binary_head`,
`test_prompt_local_v15`, `test_local_v15_stereo`. These bind to OUR output schema, so
they validate independently of the real checkpoint.

Conversion + run (user hardware, the real end-to-end gate; needs the ~5 GB v1.5 +
the v2 tokenizer + torch):

```bash
# 1. Model (auto-detects the GPT-J v1.5 arm from config shape; bf16 ok;
#    --strict must report 0 unmapped source keys).
python scripts/convert_moss_tts_local_to_gguf.py \
    --model OpenMOSS-Team/MOSS-TTS-Local-Transformer-v1.5 \
    --out models/moss-tts-local-v1_5-f32.gguf --strict
# 2. Codec: MOSS-Audio-Tokenizer-v2 via the (config-driven) Nano converter.
python scripts/convert_audio_tokenizer_nano_to_gguf.py \
    --model OpenMOSS-Team/MOSS-Audio-Tokenizer-v2 \
    --out models/moss-audio-tokenizer-v2-f32.gguf --strict
# 3. Tokenizer: v1.5 ships a standard BPE; special-token ids resolve by name.
python scripts/convert_tokenizer.py \
    --src OpenMOSS-Team/MOSS-TTS-Local-Transformer-v1.5 \
    --out models/moss-tokenizer-v1_5.gguf
# 4. Synthesize (48 kHz stereo wav) + per-stage bench.
./build/bin/moss-tts-cli tts-local --model models/moss-tts-local-v1_5-f32.gguf \
    --codec models/moss-audio-tokenizer-v2-f32.gguf \
    --tokenizer models/moss-tokenizer-v1_5.gguf --text "..." --out out.wav
./build/bin/moss-tts-cli bench local --model models/moss-tts-local-v1_5-f32.gguf \
    --codec models/moss-audio-tokenizer-v2-f32.gguf \
    --tokenizer models/moss-tokenizer-v1_5.gguf --text "..."

# 5a. VERIFY offline (always-run op-level gate; no real model needed).
ctest --test-dir build -R test_local_v15_engine --output-on-failure

# 5b. VERIFY real-model (the parity gate that proves the port on real weights).
#     The dumper loads the real modeling via trust_remote_code, written for
#     transformers 4.57.1. The .venv may hold transformers 5.x, which can break it,
#     so pin 4.57 FIRST (converters are numpy/torch-only and do not need this).
.venv/bin/pip install "transformers~=4.57"
.venv/bin/python scripts/gen_local_v15_reference.py \
    --hf-model OpenMOSS-Team/MOSS-TTS-Local-Transformer-v1.5 --out ref_v15.gguf
MOSS_TTS_LOCAL_V15=models/moss-tts-local-v1_5-f32.gguf MOSS_LOCAL_V15_REF_DUMP=ref_v15.gguf \
    ctest --test-dir build -R test_local_v15_parity --output-on-failure
#     PASS = verified. On failure, the printed per-stage maxerr (global_hidden /
#     local_text / audio / codes) localizes which stage diverged; fix + re-run.

# 5c. VERIFY the v2 codec decode against the real model (dequant + interleaved
#     waveform). Same "transformers~=4.57" pin as 5b (the dumper loads the real
#     modeling via trust_remote_code); install it first if not already done.
.venv/bin/python scripts/gen_codec_v2_reference.py \
    --hf-model OpenMOSS-Team/MOSS-Audio-Tokenizer-v2 --out ref_codec_v2.gguf
MOSS_CODEC_V2=models/moss-audio-tokenizer-v2-f32.gguf MOSS_CODEC_V2_REF=ref_codec_v2.gguf \
    ctest --test-dir build -R test_codec_v2_parity --output-on-failure
#     PASS = the RLFQ dequant + 12 decoder stages + stereo interleave match; a
#     diverging stage shows up in the per-stage maxerr.

# 6. Publish (Batch B; quantize f16/q8_0/q4_k_m + hf upload; --dry-run previews).
scripts/publish_moss_gguf.sh \
    --model models/moss-tts-local-v1_5-f32.gguf \
    --codec models/moss-audio-tokenizer-v2-f32.gguf \
    --tokenizer models/moss-tokenizer-v1_5.gguf \
    --card scripts/moss_tts_local_v1_5_card.md \
    --repo mudler/MOSS-TTS-Local-Transformer-v1.5-GGUF
```

Codec-v2 decoded-audio parity (RLFQ vs our ResidualLFQ) and HF publish stay Batch B
(separate go-ahead); high-confidence same family, but real-weights parity is
user-gated.

## V3: MossTTSRealtime (RQ-Transformer text-to-speech, offline)

The third generative variant. Same text(+reference) -> speech contract as V1/V2,
and the same RQ-Transformer shape as V2 (a **global** Qwen3 backbone over *time*,
a small **local depth** transformer over the codebooks per frame), but tuned for
the realtime model: **17 channels** (1 text + 16 audio), a **RoPE-on** depth
transformer with a **per-frame KV cache**, **per-codebook heads only**, and a
**hierarchical text-lead prompt**. The audio is decoded through the Foundation
codec's **first-16** partial-depth decode. This port is the **offline** loop —
the full model run end to end; native streaming / multi-turn is a follow-on (see
below). Generation stops on `audio_end` (EOS) on codebook 0.

### Pipeline (one synthesis)

```
text + instruction/language + (reference wav)
  -> de.tokenizer (Qwen3 BPE)        : text -> token ids
  -> prompt_rt builder (hierarchical) : chat template + (codec.encode of the
                                        reference wav, first 16 codes/frame)
                                        spliced in; text leads the audio stream
                                        (BOS_AUDIO=1025 seeded on channel 1 at the
                                        last text position); 17-row input grid +
                                        a remaining-text stream fed one id/frame
  -> RtEmbeddings (17-sum)            : per-channel table lookup over 17 tables
                                        (rt.embed.0 text + rt.embed.1..16 audio),
                                        SUMMED into one hidden/step
  -> global Qwen3 backbone (+KV cache): RoPE on; prefill the prompt then
                                        autoregressive decode, one timestep/step,
                                        producing a global hidden h_t
  -> per timestep, the DEPTH LOOP (the time x depth keystone, rt_local):
       depth 0 input = h_t INJECTED DIRECTLY (NO projection; requires
                       global_hidden == local_hidden == 2048)
       local.reset()                  : per-frame KV cache reset each frame
       for codebook i = 0..15:
         rt_local.step(in, i)         : 4-layer RoPE depth transformer, one new
                                        position appended to the per-frame KV
                                        cache (RoPE position = i), q/k-norm kept
         rt_heads.logits(i, h)        : the per-codebook head i (no head norm, no
                                        pad mask — RT samples BOS/EOS freely)
         sample codebook i            : windowed audio rep-penalty 1.1, window 50
         RtEmbeddings.embed_local_one : re-embed the just-sampled code via LOCAL
                                        table i (the off-by-one: table i feeds
                                        depth i+1) as the next depth's input
  -> stop when codebook 0 emits EOS_AUDIO (1026); drop that final frame
  -> 16 audio-code streams            -> codec.decode(n_quantizers=16) -> 24 kHz
  -> loudness_normalize                : RMS -> -20 dBFS (same as V1/V2)
```

Voice cloning runs the reference wav through `codec.encode`, keeps the **first 16
of 32** codes per frame, and splices them into the prompt grid (same spirit as
V1/V2, on the hierarchical grid).

### New `src/` files and their roles

```
qwen3.{hpp,cpp}            REUSED again: global = RoPE on; the local depth
                           transformer is ALSO RoPE-on here (unlike V2's no-RoPE
                           local) — same layer code, gated by config.
rt_local.{hpp,cpp}         the 4-layer RoPE depth transformer with a PER-FRAME KV
                           cache: reset()/step(in, depth) appends one position
                           (RoPE position = depth) instead of recomputing the
                           whole channel seq.
rt_embeddings.{hpp,cpp}    the 17 global sum-embeddings (embed_sum, prefill +
                           per-frame) AND the 15 local single-code re-embeds
                           (embed_local_one, inside the depth loop).
rt_heads.{hpp,cpp}         the 16 per-codebook LM heads (bias-free, NO head norm,
                           NO pad mask — RT may sample BOS/EOS).
prompt_rt.{hpp,cpp}        the hierarchical text-lead prompt builder + cloning
                           splice; returns prefill_ids (S x 17) + remaining_text.
moss_tts_rt.{hpp,cpp}      RealtimeTTS orchestrator: load + tts(); the pimpl
                           behind moss::Realtime in include/moss_tts.h.
rt_constants.hpp           RVQ=16, CHANNELS=17, AUDIO_VOCAB=1027, AUDIO_PAD=1024,
                           BOS_AUDIO=1025, EOS_AUDIO=1026, REF_AUDIO_PAD=151654,
                           TEXT_PAD=151655, SAMPLE_RATE=24000, REP_WINDOW=50.
```

The codec gained a partial-depth decode (`decode(..., n_quantizers=k)`) so the
16 generated codebooks decode through the 32-codebook Foundation codec (first-16,
the `n_quantizers` extension validated by `test_partial_decode`). `moss::Realtime`
(public API) is a thin pimpl over `RealtimeTTS`. CLI: `moss-tts-cli tts-rt --model
RT --codec CODEC --tokenizer TOK --text "..." [--reference R.wav] [--seed N]
[--greedy] [--language L] [--instruction I] --out OUT.wav`.

Note depth-0 injection requires **global_hidden == local_hidden == 2048** (the
hidden is fed straight into the depth transformer with no adapter); the local
transformer's **head_dim (128) != local_hidden (2048)** — head_dim/n_heads/
n_kv_heads are DERIVED FROM THE REAL local q/k_proj shapes (head_dim=128,
n_heads=16, n_kv_heads=8), not assumed.

### Converter

`scripts/convert_moss_tts_rt_to_gguf.py` -> one GGUF with four families
(verified against the real checkpoint header — 403 tensors, single shard, 0
unmapped):

- `qwen3.*` — the **global** backbone (`language_model.*` -> tensors +
  `qwen3.{hidden,n_layers,n_heads,n_kv_heads,head_dim,intermediate,rope_base,
  rms_eps,text_vocab}`). The frozen `language_model.embed_tokens.weight` is
  **intentionally skipped** (the global input path uses the summed `rt.embed.*`,
  whose `rt.embed.0` is the text path — there is no `qwen3.token_embd`).
- `rt.embed.{0..16}.weight` — the **17 global sum-embeddings**
  (`embed_tokens.{0..16}`; [0]=151936x2048 text, [1..16]=1027x2048 audio).
- `rtl.*` — the **local depth** transformer (`local_transformer.model.*` ->
  `rtl.blk.{i}.*`, `rtl.output_norm`). Its `head_dim`/`n_heads`/`n_kv_heads` are
  **DERIVED FROM THE REAL local q/k_proj shapes** (q_proj rows / head_dim =
  n_heads; k_proj rows / head_dim = n_kv_heads; head_dim = q_norm length): for
  this checkpoint head_dim=128, n_heads=16, n_kv_heads=8, local_hidden=2048.
- `rtl.embed.{0..14}.weight` (15 local input embeds, 1027x2048) and
  `rtl.head.{0..15}.weight` (16 bias-free local LM heads, 1027x2048).

Plus `rt.*` scalars / special-token ids (`rt.rvq`, `rt.audio_vocab`,
`rt.audio_pad`, `rt.bos_audio`, `rt.eos_audio`, `rt.ref_audio_pad`, `rt.text_pad`,
`rt.delay_tokens`, `rt.sample_rate`, `rt.{pad,im_start,im_end}_token_id`). Run
with `--strict` (0 unmapped against the real checkpoint).

Quant allowlist (`scripts/quantize_gguf.py`, auto-detects the kind from `qwen3.*`
+ `rtl.*`/`rt.*` tensors): QUANTIZE only the matmul weights of **both**
transformers — `qwen3.blk.{i}.attn_{q,k,v,o}` / `ffn_{gate,up,down}` and
`rtl.blk.{i}.attn_{q,k,v,o}` / `ffn_{gate,up,down}` (the only tensors read through
`ggml_mul_mat`, which dequantizes natively). KEEP F32: **all `rt.*` /
`rtl.embed.*` / `rtl.head.*` tensors** (read as raw float32 — host-staged via
`read_tensor_f32`, gathered/dotted host-side — so quantizing corrupts them) and **all `*_norm.weight`** (q/k-norm,
attn/ffn norm, both `output_norm`s).

### Tests

CI parity fixtures (model-independent; run anywhere):

| Test                  | Pins                                                    |
| --------------------- | ------------------------------------------------------ |
| `test_rt_local`        | one RoPE depth layer + per-frame KV step (q/k-norm)   |
| `test_rt_embeddings`   | the 17-sum AND 15-single (off-by-one) re-embed        |
| `test_rt_heads`        | the 16 per-codebook heads (no head norm, no pad mask) |
| `test_rt_depth_loop`   | **keystone**: the time x depth tiny-complete frame    |
| `test_prompt_rt`       | the hierarchical text-lead prompt + cloning splice    |
| `test_partial_decode`  | the codec first-k (n_quantizers) partial-depth decode |

Env-gated real-model gates (SKIP/77 without their env vars):

| Test                  | Needs                                                       |
| --------------------- | ----------------------------------------------------------- |
| `test_rt_parity`       | `MOSS_TTS_RT` + `MOSS_RT_REF_DUMP` (global+local logit parity) |
| `test_e2e_rt`          | `MOSS_TTS_RT` + `MOSS_TTS_TOKENIZER` + `MOSS_DE_TOKENIZER`  |
| `test_closed_loop_rt`  | the e2e three + `MOSS_PARAKEET_CLI` + `MOSS_PARAKEET_MODEL` |

`test_e2e_rt` runs the full V3 stack and checks the wav is 24 kHz, > 0.3 s,
audible, and unclipped. `test_closed_loop_rt` synthesizes "the quick brown
fox..." (seed 12345), shells out to a parakeet ASR CLI (verified contract:
`<cli> transcribe --model M --input W` -> transcript on stdout; the test is
env-gated so it SKIPs without the parakeet vars), then asserts >= 70% input-word
recall.

### V3 gotchas (these will bite again)

1. **RoPE-on local attention + a per-frame KV cache.** Like V2's local (which also
   keeps a per-frame KV cache, but no-RoPE), the V3 depth transformer turns RoPE ON (RoPE
   position = depth) and keeps a **per-frame KV cache that is reset every frame**
   (`local.reset()` before each timestep's depth loop). Forgetting the reset
   leaks codebook history across frames.
2. **Depth-0 injection has NO projection.** The backbone hidden `h_t` is fed
   straight into the depth transformer as the depth-0 input — there is no adapter
   MLP. This requires **global_hidden == local_hidden == 2048**; do not add a
   projection or it will not match the checkpoint.
3. **Off-by-one local embed index.** There are **15** local input embeds and
   **16** heads. Local table `i` (the codebook just produced) feeds depth `i+1`;
   head `i` reads depth `i`. The last codebook (15) has a head but no local
   re-embed (nothing follows it).
4. **Per-codebook heads ONLY — no head norm, no pad mask.** Unlike V2 (head_norm
   + pad masking), the RT heads are a bare per-codebook `lm_head`; the sampler may
   freely produce BOS_AUDIO/EOS_AUDIO (that's how the loop stops). Don't mask
   them out.
5. **rvq=16, decoded via the codec's first-16 partial decode.** The model emits
   16 codebooks; they decode through the 32-codebook Foundation codec using the
   `n_quantizers=16` partial-depth extension (`Codec::decode(..., 16)`). The codec
   must report `num_quantizers() >= 16` at load (checked).
6. **Stop on audio EOS on codebook 0.** Generation halts when codebook 0 samples
   `EOS_AUDIO` (1026); that final frame is recorded for the step count then
   dropped before decode.
7. **Windowed rep-penalty 1.1, window 50.** The audio sampler defaults to penalty
   1.1 but only over the **last 50** history codes per codebook
   (`REP_WINDOW=50`, upstream `inferencer.py ht = ht[:, -repetition_window:]`).

### KNOWN FOLLOW-UPS (V3)

1. **Native streaming / multi-turn is deferred.** This port is the **offline**
   loop — the complete model run, text in / wav out. The realtime model's native
   streaming + multi-turn conversation path is a follow-on; the offline loop is
   the full model and is what the gates validate.
2. **`prompt_rt` fixture provenance.** The `test_prompt_rt` fixture was dumped
   from a **faithful NUMPY RE-DERIVATION** of the Realtime processor (torch was
   unavailable in the venv), **not** the genuine inferencer — including the
   `id 151654` (REF_AUDIO_PAD) splice. Recommend a one-time cross-check against
   the real inferencer on a torch box on the first real-model run, then re-bless
   the fixture if it drifts.
3. **GPU `->data` path — DONE.** Embeddings / heads host-stage their weight tables
   via `moss::read_tensor_f32`/`read_tensor_i32` at load and `local_adapters` uses
   the device-safe graph-with-inputs pattern (CPU byte-identical; `src/` grep-clean
   of tensor `->data`); a GPU build is correct-by-construction. See the V1 follow-up
   for the full description + residuals (GPU-unverified here; host-RAM duplication;
   category-B ggml-op unit tests).

### Closing V3 on real hardware

```bash
# 1. Convert the Realtime backbone + the BPE tokenizer (codec = the Foundation's,
#    shared with V1/V2; no separate codec conversion needed).
.venv/bin/python scripts/convert_moss_tts_rt_to_gguf.py \
    --model OpenMOSS-Team/MOSS-TTS-Realtime \
    --out models/moss-tts-rt-f32.gguf --strict
.venv/bin/python scripts/convert_tokenizer.py --model OpenMOSS-Team/MOSS-TTS \
    --out models/moss-de-tokenizer.gguf

# 2. Produce the global+local logit-parity reference dump
.venv/bin/python scripts/gen_rt_reference.py \
    --model OpenMOSS-Team/MOSS-TTS-Realtime --out /tmp/rt_ref.gguf

# 3. Wire env vars + run the env-gated gates
export MOSS_TTS_RT=models/moss-tts-rt-f32.gguf
export MOSS_RT_REF_DUMP=/tmp/rt_ref.gguf
export MOSS_TTS_TOKENIZER=models/moss-audio-tokenizer-f32.gguf
export MOSS_DE_TOKENIZER=models/moss-de-tokenizer.gguf
ctest --test-dir build -R 'test_rt_parity|test_e2e_rt' --output-on-failure

# 4. Closed-loop WER gate needs a parakeet.cpp build + ASR model
export MOSS_PARAKEET_CLI=/path/to/parakeet-cli
export MOSS_PARAKEET_MODEL=/path/to/parakeet-asr.gguf
ctest --test-dir build -R test_closed_loop_rt --output-on-failure

# 5. Benchmark wall-time + RTF
./bench_rt.sh cpu --model models/moss-tts-rt-f32.gguf \
    --codec models/moss-audio-tokenizer-f32.gguf --tokenizer models/moss-de-tokenizer.gguf
```

## V4: MossTTSNano (GPT-2+RoPE RQ-Transformer, 100M, 48kHz stereo)

The fourth and FINAL generative variant — porting V4 completes the WHOLE
OpenMOSS MOSS-TTS family to native ggml. Same text(+reference) -> speech
contract as V1/V2/V3, and the same two-tier RQ-Transformer shape (a **global**
backbone over *time*, a small **local depth** transformer over the codebooks per
frame), but the 100M Nano model swaps the backbone family: both tiers are
**GPT-2 + interleaved RoPE** (not Qwen3), the frame is a **17-wide flat AR row
stream** (col0=text, cols1..16=16 audio codebooks), the stop is a **decision
token** at depth 0, the codec is a **48kHz STEREO "Cat" codec** decoded
streaming, the tokenizer is **native SentencePiece-unigram**, and the *entire*
target text is **prefilled** (NO text streaming). Output is **interleaved stereo
@ 48000 Hz**.

### Pipeline (one synthesis)

```
text + (reference wav)
  -> text_cleanup                     : robust NON-semantic cleanup (whitespace,
                                        controls) — WeText TN is deferred
  -> SpTokenizer (SentencePiece-unigram, native): text -> token ids (Viterbi over
                                        the SP-normalized byte string, byte-fallback)
  -> prompt_nano builder              : assembles the (S, 17) prefill ROW stream —
                                        col0 = text ids, cols1..16 = audio codes;
                                        text rows pad cols1..16 with AUDIO_PAD
                                        (1024); reference rows put AUDIO_USER_SLOT
                                        (8) on col0 + the 16 reference codes.
                                        The ENTIRE target text is prefilled (no
                                        remaining-text stream).
  -> NanoEmbeddings (17-sum)          : per-channel table lookup over 17 tables
                                        (nano.embed.0 text + nano.embed.1..16
                                        audio), SUMMED into one hidden/step.
                                        audio_pad (1024) masks to a ZERO embedding
                                        (the audio tables are 1024-row, NO pad row).
  -> global GPT-2 backbone (+KV cache): 12 layers, interleaved RoPE (NORMAL mode),
                                        LayerNorm-with-bias, fused c_attn, gelu_new
                                        MLP, MHA no-GQA; prefill then autoregressive
                                        decode one timestep/step -> global hidden h_t
  -> per timestep, the DEPTH LOOP (the time x depth keystone, nano_local + heads):
       local.reset()                  : per-frame KV cache reset each frame
       depth 0 input = h_t INJECTED DIRECTLY (NO projection; global_hidden ==
                       local_hidden == 768)
       depth 0 -> nano_local.step(h_t, 0) -> the TEXT head -> DECISION logits;
         sample the decision token. STOP iff decision != AUDIO_ASSISTANT_SLOT (9).
         Else re-embed the decision via the TEXT table -> depth-1 input.
       for codebook c = 0..15:
         depth d=c+1 -> nano_local.step(in, d) (1-layer RoPE GPT-2, RoPE pos = d)
         heads.audio_logits(c, h)     : per-codebook audio head c (tied to the
                                        audio embed c)
         sample code c                : audio rep-penalty over the full per-codebook
                                        history (unbounded, matching upstream Nano)
         emb.embed_audio_one(c, code) : re-embed the just-sampled code via audio
                                        table c (off-by-one: table c feeds depth c+1)
  -> next global step: next_ids = {AUDIO_ASSISTANT_SLOT, code_0..code_15}
  -> 16 audio-code streams            -> codec.decode (RVQ-16) -> 48 kHz STEREO,
                                        streaming per-stage ring-KV decode
  -> loudness_normalize (stereo)       -> interleaved stereo wav @ 48000 Hz
```

Depth 0 is the one wiring subtlety: the **decision token is read off the TEXT
head** (the text vocab table), and STOP is `decision != AUDIO_ASSISTANT_SLOT(9)`
(continue-decision) — it is NOT a separate EOS codebook-0 token like V3. col0 of
every *generated* frame is `AUDIO_ASSISTANT_SLOT`. Voice cloning runs the
reference wav through `codec.encode` and splices the 16 codes/frame into the
reference rows of the prompt grid.

### New `src/` files and their roles

```
gpt2.{hpp,cpp}            the shared GPT-2 + interleaved-RoPE layer, reused by
                          BOTH tiers: LayerNorm-WITH-bias, fused c_attn (one
                          matmul -> 3H + bias, split q/k/v), gelu_new MLP (c_fc ->
                          ggml_gelu tanh-approx -> mlp_cproj, NO SwiGLU), MHA
                          no-GQA, interleaved RoPE == GGML_ROPE_TYPE_NORMAL
                          (GPT-J pairing of dims 2i,2i+1).
nano_backbone.{hpp,cpp}   the 12-layer GLOBAL GPT-2 backbone over time + a
                          persistent KV cache (prefill + per-step decode).
nano_local.{hpp,cpp}      the 1-layer LOCAL/depth GPT-2 transformer with a
                          per-frame KV cache: reset()/step(in, depth) appends one
                          position (RoPE pos = depth).
nano_embeddings.{hpp,cpp} the 17 input tables (embed_sum prefill/per-frame;
                          embed_text_one for the decision re-embed; embed_audio_one
                          for the depth-loop code re-embeds). Audio tables are
                          1024-row (no pad row); AUDIO_PAD masks to zero.
nano_heads.{hpp,cpp}      the 1 text head (the decision) + 16 per-codebook audio
                          heads (all TIED to their embedding source).
sp_tokenizer.{hpp,cpp}    native SentencePiece-unigram tokenizer (Viterbi over
                          the SP-normalized byte string, byte-fallback / <unk>);
                          NO sentencepiece runtime dep.
text_cleanup.{hpp,cpp}    robust NON-semantic text cleanup (whitespace/control
                          normalization). WeText semantic TN is deferred.
prompt_nano.{hpp,cpp}     the (S, 17) row-stream prefill builder + cloning splice;
                          returns prefill_ids (S x 17) and remaining_text (empty
                          for Nano — the whole text is prefilled).
moss_tts_nano.{hpp,cpp}   NanoTTS orchestrator: load + tts()/tts_stream(); the
                          pimpl behind moss::Nano in include/moss_tts.h.
nano_constants.hpp        N_VQ=16, CHANNELS=17, AUDIO_VOCAB=1024, AUDIO_PAD=1024,
                          TEXT_VOCAB=16384, PAD=3, IM_START=4, IM_END=5,
                          AUDIO_START=6, AUDIO_END=7, AUDIO_USER_SLOT=8,
                          AUDIO_ASSISTANT_SLOT=9, SAMPLE_RATE=48000, N_CHANNELS=2.
audio_io.{hpp,cpp}        stereo additions: save_wav(...,n_channels),
                          load_wav_stereo, resample_linear, stereo loudness.
audio_tokenizer.cpp       stereo/streaming additions: channel-interleave I/O-fold
                          (channels=2, factor 2), the streaming per-stage ring-KV
                          decode_step, and the T14 sliding-window context fix.
```

`moss::Nano` (public API) is a thin pimpl over `NanoTTS`. CLI: `moss-tts-cli
tts-nano --model NANO --codec CODEC --tokenizer TOK --text "..." [--reference
R.wav] [--seed N] [--greedy] [--language L] [--instruction I] [--stream] --out
OUT.wav`.

### Converters

`scripts/convert_moss_tts_nano_to_gguf.py` -> one GGUF (verified against the
published HF modeling: `MossTTSNanoForCausalLM`, `gpt2_decoder.py`):

- `gpt2.*` — the **global** 12-layer backbone (`transformer.h.{i}.*` ->
  `gpt2.blk.{i}.{ln1,cattn,cproj,ln2,cfc,mlp_cproj}`, `gpt2.output_norm`).
  Every projection is a plain **`nn.Linear` ([out,in])** — NOT an HF GPT-2
  `Conv1D` — so weights pass through with NO transpose (a Conv1D transpose would
  be WRONG here). The MLP is `mlp.fc_in`/`mlp.fc_out` (GPT-J naming) ->
  `cfc`/`mlp_cproj`. `transformer.wte` -> `nano.embed.0` (it is the text path;
  there is no separate `gpt2.token_embd`).
- `gptl.*` — the **local** 1-layer depth transformer (`local_transformer.h.0.*`,
  identical per-layer layout; `local_transformer.wte` is a frozen `nn.Identity`
  placeholder and is SKIPPED).
- `nano.embed.{0..16}.weight` — the 17 input tables (`[0]`=16384x768 text =
  `transformer.wte`; `[1..16]`=1024x768 audio = `audio_embeddings.{0..15}`,
  **1024-row, NO pad row** — the model masks AUDIO_PAD to a zero embedding).
- `nano.head.text.weight` + `nano.head.audio.{0..15}.weight` — TIED to wte /
  audio_embeddings; the saved state-dict typically OMITS them, so the converter
  SYNTHESIZES them from the tied source (and SKIPs a redundant explicit head key
  if present).

`scripts/convert_audio_tokenizer_nano_to_gguf.py` -> the **48kHz STEREO Cat
codec** GGUF (same `moss.at.*` schema as the Foundation codec, distinct file):
`number_channels=2`, `enable_channel_interleave` (the 2 channels fold into the
codec sequence, factor 2), **multi-stage** generic encoder/decoder towers
(PatchedPretransform reshapes + ProjectedTransformer stages with LayerScale +
RoPE), RVQ-16, code_dim=768. Adds `moss.at.channels` (2) +
`moss.at.channel_interleave`. ResidualLFQ in/out projections are WNConv1d ->
fused into dense linears (`fuse_wn`, same as Foundation).

`scripts/convert_tokenizer.py` gained the **SentencePiece path** (SP-unigram ->
the tokenizer gguf the C++ `SpTokenizer` reads).

Quant allowlist (`scripts/quantize_gguf.py`, auto-detects from `gpt2.*`/`gptl.*`/
`nano.*`): QUANTIZE only the matmul weights of both transformers —
`gpt2.blk.{i}.{cattn,cproj,cfc,mlp_cproj}.weight` and the matching
`gptl.blk.{i}.*` (the only tensors read through `ggml_mul_mat`). KEEP F32: **all
`nano.embed.*` / `nano.head.*`** (read raw f32 — host-staged via `read_tensor_f32`,
gathered/dotted host-side), **all `*.bias`** and
**all `*output_norm.*` / `ln1.* / ln2.*` norms**.

### Tests

CI parity fixtures (model-independent; run anywhere):

| Test                  | Pins                                                       |
| --------------------- | ---------------------------------------------------------- |
| `test_gpt2_layer`      | one GPT-2 + interleaved-RoPE (NORMAL) layer               |
| `test_nano_backbone`   | the 12-layer global backbone + persistent KV              |
| `test_nano_local`      | one local depth layer + per-frame KV step                 |
| `test_nano_embed`      | the 17-sum + single-token re-embeds (off-by-one)          |
| `test_nano_heads`      | the 1 text + 16 tied audio heads                          |
| `test_sp_tokenizer`    | the native SentencePiece-unigram Viterbi (tiny fixture)   |
| `test_text_cleanup`    | the robust non-semantic text cleanup                      |
| `test_prompt_nano`     | the (S, 17) row-stream prefill + cloning splice           |
| `test_nano_frame_loop` | **keystone**: the time x depth tiny-complete frame        |
| `test_nano_codec`      | the 48kHz stereo Cat codec decode                         |
| `test_nano_codec_stream`| the streaming per-stage ring-KV == offline decode        |
| `test_audio_io_stereo` | stereo save/load + resample + loudness                    |

Env-gated real-model gates (SKIP/77 without their env vars):

| Test                   | Needs                                                      |
| ---------------------- | --------------------------------------------------------- |
| `test_nano_parity`      | `MOSS_TTS_NANO` + `MOSS_NANO_REF_DUMP` (global+local parity) |
| `test_e2e_nano`         | `MOSS_TTS_NANO` + `MOSS_NANO_CODEC` + `MOSS_NANO_TOKENIZER` |
| `test_nano_stream_e2e`  | the e2e three (streaming-vs-offline equivalence)          |
| `test_closed_loop_nano` | the e2e three + `MOSS_PARAKEET_CLI` + `MOSS_PARAKEET_MODEL` |

`test_e2e_nano` runs the full V4 stack and checks the wav is 48 kHz, interleaved
STEREO, > 0.3 s, audible, and unclipped. `test_nano_stream_e2e` asserts the
`tts_stream` accumulation EQUALS the offline `tts` buffer for the same seed/text
(the streaming codec path is exact). `test_closed_loop_nano` synthesizes "the
quick brown fox..." (seed 12345), **downmixes 48k stereo -> 16k mono** (parakeet
is 16k mono), shells out to a parakeet ASR CLI (`<cli> transcribe --model M
--input W` -> transcript on stdout), then asserts >= 70% input-word recall.

### V4 gotchas (these will bite again)

1. **Interleaved RoPE == GGML_ROPE_TYPE_NORMAL (mode 0).** Both GPT-2 tiers pair
   dims `2i,2i+1` (GPT-J style); use the NORMAL ggml rope mode, NOT NEOX. This is
   confirmed against the `w_rope` / `w_gpt2_block` fixtures.
2. **GPT-2 here is `nn.Linear`, NOT Conv1D — no transpose.** Every projection is
   stored `[out,in]` already. Do NOT apply the HF GPT-2 Conv1D transpose.
3. **Tied heads.** The text/audio heads are tied to their embedding tables; the
   converter synthesizes them when the state-dict omits them.
4. **Audio embeds are 1024-row, pad-masked (NO pad row).** AUDIO_PAD (1024)
   never indexes a row — it masks to a zero embedding. Don't add a 1025th row.
5. **Decision-via-the-TEXT-head + decision != AUDIO_ASSISTANT_SLOT(9) stops.**
   Depth 0 reads the TEXT head for the continue/stop decision (not an audio EOS
   codebook); the decision token is re-embedded via the TEXT table for depth 1.
6. **Depth-0 injection has NO projection** (global_hidden == local_hidden == 768).
7. **Off-by-one audio re-embed.** Audio table `c` feeds depth `c+1`; head `c`
   reads depth `c+1`. The last codebook has a head but no following re-embed.
8. **Stereo channel-interleave I/O-fold.** channels=2 with channel_interleave
   folds the 2 channels into the codec sequence (factor 2); the decode emits
   interleaved stereo. The mono path (channels==1) stays byte-identical.
9. **The decoder sliding-window context uses the INTERLEAVED PATCH PRODUCT, not
   the per-channel downsample (the T14 fix).** For interleaved stereo the patch
   product is `downsample_ * factor`; seed the sliding window from the full
   encoder patch product, NOT just `downsample_`, or the window context drifts.
10. **Streaming per-stage ring-KV is EXACT only because the codec is RoPE-only.**
    There is no additive sin/positional bias, so a per-stage ring-KV (cached K/V
    evicted to each stage's `context`) reproduces the offline output bit-for-bit.
    A non-RoPE codec variant would BREAK this — don't assume streaming
    equivalence for any future additive-positional codec.
11. **SP-unigram NFKC is an APPROXIMATION.** Normalization is minimal
    (whitespace-collapse + add_dummy_prefix + `▁`); the heavy NFKC rule set is
    NOT applied. Exact for ASCII; adequate for typical Nano text.
12. **prompt_nano uses the REAL prompt template.** The manifest `prompt_templates`
    token-id arrays (user_prompt_prefix / user_prompt_after_reference /
    assistant_prompt_prefix) are now built by tokenizing the literal template TEXT
    via the SP tokenizer at build time (`tok.encode(...)` in
    `build_generation_prompt_nano`):
    `[IM_START] ++ enc("user\n") ++ enc("<user_inst>\n- Reference(s):\n")`,
    `enc("\n- Instruction:\nNone\n...\n- Text:\n")`, and
    `enc("\n</user_inst>") ++ [IM_END] ++ enc("\n") ++ [IM_START] ++ enc("assistant\n")`.
    The no-clone path substitutes the literal `enc("None")` for the
    AUDIO_START/reference/AUDIO_END framing.
    PROVENANCE is the upstream HF **INFERENCE** builder — `prompting.py`
    (`build_user_prompt_prefix` / `build_user_prompt_after_reference` /
    `build_assistant_prompt_prefix` / `build_prompt_prefix`) and
    `modeling_moss_tts_nano.py::build_inference_input_ids` (voice-clone +
    continuation branches). The per-segment encode granularity (SEPARATE
    `tok.encode(...)` per template segment, concatenated) MATCHES inference
    (`prompting.py`'s per-segment `encode_text` calls), NOT the training
    `finetuning/dataset.py` (which joint-encodes) — so dataset.py is the wrong
    provenance to cite.
    The fixture tokenizes the IDENTICAL text through the same tiny SP vocab, so
    `test_prompt_nano` parity stays exact.
    CAVEAT: prompt STRUCTURE + per-segment encode granularity are now
    inference-faithful, but the EXACT per-segment token boundaries on the real
    model still depend on SentencePiece `add_dummy_prefix`/normalization fidelity
    (whether the native SpTokenizer's per-segment leading-`▁` matches the real
    `tokenizer.model` normalizer). That exactness rolls up into the existing
    SP-fidelity caveat (gotcha 11 / follow-up 3) — confirm on a real-tokenizer
    cross-check.

### KNOWN FOLLOW-UPS (V4)

1. **WeText semantic normalization deferred.** Text cleanup is non-semantic only;
   number/abbreviation TN (WeText) is a follow-on.
2. **prompt_nano template structure is inference-faithful** (from upstream HF
   INFERENCE `prompting.py` + `modeling_moss_tts_nano.py::build_inference_input_ids`;
   see gotcha 12) — DONE, tokenized via SP at build time. The one open piece is
   the EXACT per-segment SP token boundaries on the real model, which depend on
   `add_dummy_prefix`/normalization fidelity and roll up into the SP-fidelity
   follow-up (#3) — confirm on a real-tokenizer cross-check. Re-bless only if the
   upstream template text itself changes.
3. **SP NFKC approximation** (gotcha 11) — full NFKC if real text needs it; this
   same caveat covers the per-segment leading-`▁`/`add_dummy_prefix` exactness the
   prompt_nano template (#2) depends on, to confirm against the real
   `tokenizer.model`.
4. **Codec ENCODE exact-parity is validated** — `test_nano_codec` pins the FULL
   stereo encode path (waveform → encoder tower [patch_down + Cat transformer
   stages] → quantizer residual-VQ argmax loop → codes) by EXACT integer match
   against a committed numpy reference (`ref.enc_wav` → `ref.enc_codes` in the
   `w_nano_codec` fixture, mirroring `quantize`/`encode` op-for-op with tie-free
   similarities so ggml argmax-last == numpy argmax-first). Decode parity is
   likewise validated by `test_nano_codec` / `_stream`.
5. **Streaming codec decode is true per-stage ring-KV** — DONE. Each decoder
   transformer stage keeps a persistent per-layer host-side K/V cache (already
   RoPE'd), evicted to that stage's sliding-window `context`. Each streaming step
   runs ONE graph over the new code frame, so it is incremental (**O(context)**
   attention + O(1) projections over the new frame) instead of re-decoding the
   O(window) receptive field. Numerically identical to `decode_full` (validated by
   `test_nano_codec_stream` == `decode_full` on the every-stage-clips fixture,
   which decisively catches a wrong RoPE-offset / eviction). Still exact ONLY
   because the codec is RoPE-only (see gotcha 10).
6. **GPU `->data` path — DONE.** Embeddings / heads host-stage their weight tables
   via `moss::read_tensor_f32`/`read_tensor_i32` at load and `local_adapters` uses
   the device-safe graph-with-inputs pattern (CPU byte-identical; `src/` grep-clean
   of tensor `->data`); a GPU build is correct-by-construction. See the V1 follow-up
   for the full description + residuals (GPU-unverified here; host-RAM duplication;
   category-B ggml-op unit tests).
7. **Backbone KV cache is device-resident — DONE.** Both global backbones
   (`DelayBackbone`/Qwen3 for V1/V2/V3, `NanoBackbone`/GPT-2 for V4) previously
   kept per-layer K/V as host `std::vector`s and re-uploaded/-read-back the full
   grown cache every decode step (O(T^2) host<->device transfer; a full-KV PCIe
   roundtrip per step on GPU). Now each backbone owns a persistent
   `[head_dim, n_kv, max_seq, 1]` per-layer cache on its own backend buffer
   (n_kv = n_kv_heads for Qwen3, n_head for GPT-2 / no GQA);
   `qwen3_layer_forward`/`gpt2_layer_forward` gained an opt-in device-cache path
   (store the new columns via in-graph `ggml_cpy` to a cache view, read the
   `[0:past+T]` prefix by view) — KV data movement is O(T) and the per-step
   transfer is just the new hidden row in/out (mask is null on the T==1 decode
   step: every cached key is causally valid). Null cache => the existing additive
   `k_past`/`v_past` path, so the codec (`transformer.cpp`) and the bounded depth
   caches (`local_transformer`/`rt_local`/`nano_local`) are byte-identical.
   Validated by `test_delay_kv` / `test_nano_backbone` (decode==full-prefill at
   the same maxerr: 1.79e-7 / 6.97e-5) + revert-to-fail discrimination on the
   store offset. Residual: GPU execution still unverified on this CPU-only box
   (correct-by-construction); the depth-loop caches stay host-driven
   (small/bounded, not worth converting).

### Closing V4 on real hardware

```bash
# 1. Convert the Nano LLM, the 48k stereo codec, and the SP tokenizer.
.venv/bin/python scripts/convert_moss_tts_nano_to_gguf.py \
    --model OpenMOSS-Team/MOSS-TTS-Nano-100M \
    --out models/moss-tts-nano-f32.gguf --strict
.venv/bin/python scripts/convert_audio_tokenizer_nano_to_gguf.py \
    --model OpenMOSS-Team/MOSS-Audio-Tokenizer-Nano \
    --out models/moss-audio-tokenizer-nano-f32.gguf --strict
.venv/bin/python scripts/convert_tokenizer.py \
    --model OpenMOSS-Team/MOSS-TTS-Nano-100M \
    --out models/moss-nano-tokenizer.gguf       # SentencePiece path

# 2. Produce the global+local logit-parity reference dump
.venv/bin/python scripts/gen_nano_reference.py \
    --model OpenMOSS-Team/MOSS-TTS-Nano-100M --out /tmp/nano_ref.gguf

# 3. Wire env vars + run the env-gated gates
export MOSS_TTS_NANO=models/moss-tts-nano-f32.gguf
export MOSS_NANO_REF_DUMP=/tmp/nano_ref.gguf
export MOSS_NANO_CODEC=models/moss-audio-tokenizer-nano-f32.gguf
export MOSS_NANO_TOKENIZER=models/moss-nano-tokenizer.gguf
ctest --test-dir build -R 'test_nano_parity|test_e2e_nano|test_nano_stream_e2e' \
    --output-on-failure

# 4. Closed-loop WER gate needs a parakeet.cpp build + ASR model
export MOSS_PARAKEET_CLI=/path/to/parakeet-cli
export MOSS_PARAKEET_MODEL=/path/to/parakeet-asr.gguf
ctest --test-dir build -R test_closed_loop_nano --output-on-failure

# 5. Benchmark wall-time + RTF
./bench_nano.sh cpu --model models/moss-tts-nano-f32.gguf \
    --codec models/moss-audio-tokenizer-nano-f32.gguf \
    --tokenizer models/moss-nano-tokenizer.gguf
```

## Real-model validation (closing the milestone)

Numeric parity + benchmark numbers require the real ~7GB checkpoint + ONNX
runtime (the user's hardware). The C++ tests are env-gated and SKIP without them.

```bash
# 1. Convert the upstream safetensors -> f32 GGUF
.venv/bin/python scripts/convert_audio_tokenizer_to_gguf.py \
    --model OpenMOSS-Team/MOSS-Audio-Tokenizer \
    --out models/moss-audio-tokenizer-f32.gguf --strict

# 2. Produce the ONNX reconstruction reference for the same input clip
#    (needs the ONNX repo: encoder.onnx + decoder.onnx + .data files)
.venv/bin/pip install onnxruntime soundfile     # reference-only deps
.venv/bin/python scripts/gen_onnx_reference.py \
    --onnx-dir weights/MOSS-Audio-Tokenizer-ONNX \
    --in samples/clip.wav --out-dir /tmp/parity

# 3. Run the parity gate
export MOSS_TTS_TOKENIZER=models/moss-audio-tokenizer-f32.gguf
export MOSS_TTS_PARITY_IN=samples/clip.wav
export MOSS_TTS_PARITY_REF=/tmp/parity/ref_recon.wav
ctest --test-dir build -R test_reconstruct_parity --output-on-failure

# 4. Benchmark reconstruction RTF (drop clips into samples/*.wav)
./bench.sh cpu  models/moss-audio-tokenizer-f32.gguf
./bench.sh cuda models/moss-audio-tokenizer-f32.gguf
```

The parity SNR threshold (`kMinOnnxSnr = 20.0 dB` in
`tests/test_reconstruct_parity.cpp`) is a first guess for f32; tune it on the
first real run and loosen for quantized GGUFs. Also confirm whether the ONNX
export bakes the same sliding-window attention — if ONNX is plain-causal, our
long-audio output could differ from ONNX while still matching PyTorch; pick the
reference accordingly.

### ONNX I/O signature (verified)

Inspected via `onnx.load(..., load_external_data=False)` on
`OpenMOSS-Team/MOSS-Audio-Tokenizer-ONNX`:

```
encoder.onnx  inputs : input_values FLOAT [1,1,seq_len], n_quantizers INT64 scalar
              outputs: codes INT64 [32,1,floor(seq_len/1920)], len INT64 [1]
decoder.onnx  inputs : audio_codes INT64 [32,1,code_len],   n_quantizers INT64 scalar
              outputs: waveform FLOAT [1,1,1920*code_len],   len INT64 [1]
```

`gen_onnx_reference.py` addresses inputs by name (stable) and outputs by index
(the output names are export-mangled). The `.onnx` graph files reference external
weights in `encoder.data` / `decoder.data`; keep all four together.

## Adding a test

```bash
cp tests/test_smoke.cpp tests/test_my_thing.cpp   # return 0 = pass, 77 = skip
echo 'moss_add_test(test_my_thing)' >> tests/CMakeLists.txt
```

For an op parity test, add its reference dump to `scripts/gen_test_fixtures.py`
and load it in the test. For a real-model test, gate on the relevant
`MOSS_TTS_*` env var and `return 77` when it's unset.

## ggml submodule

Pinned in `third_party/ggml`, no local patches. To bump: update the SHA, run
`ctest --test-dir build --output-on-failure`, fix any C-API breakage in
`src/model_loader.cpp` / `src/backend.cpp`.

## Style

- C++17; no exceptions across the C-API (return status codes). Every flat
  C-API entrypoint that invokes throwing C++ (`*_load`, `*_tts`,
  `moss_nano_tts_stream`, `moss_codec_reconstruct`) wraps the call in
  `try { ... } catch (...) { <documented failure return> }` so a `std::bad_alloc`
  / parse / I/O exception can never propagate across the `extern "C"` boundary
  (UB). The catch frees any partial allocation (deletes the handle / no leak),
  zeroes the out-params to the same safe values as the normal-failure path, and
  returns the API's failure value (nullptr / nonzero). `tests/test_capi_robustness.cpp`
  pins the no-crash-on-bad-input contract (null handles + nonexistent paths) and
  runs unconditionally.
- One translation unit per logical component; keep `moss_tts.cpp` /
  `moss_tts_capi.cpp` thin — they forward into the `moss::` namespace.
- Comments explain non-obvious *why*, not *what* — the gotchas list above is the
  model for what a *why* comment looks like.

---
> Source: [mudler/moss-tts.cpp](https://github.com/mudler/moss-tts.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
