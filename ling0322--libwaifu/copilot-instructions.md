## libwaifu

> cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DWITH_OPENMP=ON -DWITH_MLX=ON

# libwaifu

## Build

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DWITH_OPENMP=ON -DWITH_MLX=ON
cmake --build build -j$(sysctl -n hw.ncpu)
```

## Test

Before committing, both fast suites must pass:

```bash
./build/unittest
LIBWAIFU_LIB_DIR="$PWD/build" cargo test --manifest-path waifu/Cargo.toml
```

`LIBWAIFU_LIB_DIR` must be absolute because the Rust build script runs from `waifu/`.

The plain `cargo test` does not compile `waifu/src/seed_vc/`, which is behind the off-by-default
`gpl` feature. When a change touches it, also run
`LIBWAIFU_LIB_DIR="$PWD/build" cargo test --manifest-path waifu/Cargo.toml --features gpl --lib seed_vc`.

## Licence

libwaifu is MIT, except the Seed-VC port (`waifu/src/seed_vc/`, `waifu/tests/seed_vc.rs`,
`waifu/examples/convert.rs`, `tools/seed_vc_*.py`), which is GPL-3.0-only and compiled only with
the `gpl` feature (CMake: `-DENABLE_GPL=ON`). Keep GPL code out of every other file, keep MIT code
from depending on `seed_vc`, and never make a default feature depend on `gpl`.

### Model tests

Model tests are ignored and take over one minute. Run them only when explicitly requested; do not
ask whether to run them. Otherwise, report that they were skipped and provide the relevant command.
Select suites by test binary with `--test`, not by name filters such as `--skip sdxl`.

```bash
# waifu/src/anima/, tools/anima_*
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test anima --test anima_pipeline --test anima_tokenizer -- --ignored --test-threads=1

# waifu/src/layers.rs, waifu/src/sdxl/, flint/
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test sdxl --test sdxl_unet --test sdxl_vae --test sdxl_text_encoder \
    --test sdxl_tokenizer --test sdxl_sampler -- --ignored --test-threads=1

# waifu/src/krea2/, waifu/src/qwen_vae.rs, waifu/src/flow.rs, tools/krea2_exporter.py
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test krea2 --test krea2_sampler --test krea2_pipeline --test krea2_tokenizer \
    -- --ignored --test-threads=1

# waifu/src/qwen_image/, tools/qwen_image_exporter.py
# qwen_image_control needs models/qwen-image-2.1-controlnet.yaml and its _test.safetensors
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test qwen_image --test qwen_image_pipeline --test qwen_image_tokenizer \
    --test qwen_image_control -- --ignored --test-threads=1

# waifu/src/cosyvoice3/, tools/cosyvoice3_*
# needs models/cosyvoice3.yaml and models/cosyvoice3_test.safetensors; docs/cosyvoice3.md says how
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test cosyvoice3 -- --ignored --test-threads=1

# waifu/src/seed_vc/, tools/seed_vc_* -- GPL-3.0, behind the `gpl` feature (docs/seed_vc.md)
# needs models/seed_vc.yaml and models/seed_vc_test.safetensors; docs/seed_vc.md says how
cargo test --release --manifest-path waifu/Cargo.toml --features gpl --no-fail-fast \
    --test seed_vc -- --ignored --test-threads=1

# waifu/src/indextts/, tools/indextts_* (campplus.rs and features.rs: run cosyvoice3's too)
# needs models/indextts25.yaml and the two recordings tools/indextts_reference_voices.py writes
cargo test --release --manifest-path waifu/Cargo.toml --no-fail-fast \
    --test indextts --test indextts_gpt -- --ignored --test-threads=1
```

- Keep `--no-fail-fast` and `--test-threads=1`; each test loads a full model onto the GPU.
- Do not add `--features cli`.
- Run only one GPU job at a time. Check `nvidia-smi` before diagnosing OOM or timing failures.
- No individual test may exceed three minutes. Reuse model fixtures across tests instead of
  loading a model per test. The harness gives every test its own thread, so a thread-local
  fixture is read once per *test* and not once per binary: a package the size of Krea 2's is
  checked in one test that asks several questions rather than several that each read it.
- A reference input must be one the model would really be handed -- a latent off a real
  trajectory, not `torch.randn`. Two implementations agreeing on activations neither was trained
  to see says nothing about the pictures they draw.

## Git

- Never modify, commit to, or push directly to `main`. Use a non-`main` branch and a pull request.
- Commit or push only when explicitly requested in the current conversation.
- Never push before the user has reviewed and requested the commit.

### Pull requests

A title and a first paragraph are what everyone else reads. Write them so that somebody scanning
the list, or opening the page for ten seconds, knows what changed.

- **The title names what changed, not what it feels like.** `module: what it does now` —
  `indextts_gpt: add KV-cached generation (prefill/step/generate)`. Not a sentence in the voice of
  the feature: "Say a sentence one token at a time" reads well and tells a reviewer neither which
  module moved nor what was added to it.
- **The first paragraph says what the change does, in one or two sentences**, and says it first.
  Name the module, the new types or functions, and anything removed. A reader who has finished
  that paragraph should be able to stop.
- **Everything else comes after it** — why it was built that way, what was tried and rejected,
  what is still missing, what the tests found. Put it under headings so it can be skipped.
- **Keep it short.** A description nobody finishes is a description that did not say anything.

---
> Source: [ling0322/libwaifu](https://github.com/ling0322/libwaifu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
