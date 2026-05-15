## deepvqe-ggml

> GGML inference and PyTorch training for DeepVQE (Indenbom et al.,

# DeepVQE-GGML

GGML inference and PyTorch training for DeepVQE (Indenbom et al.,
Interspeech 2023) — joint acoustic echo cancellation, noise suppression,
and dereverberation.

**Paper**: [DeepVQE: Real Time Deep Voice Quality Enhancement](https://arxiv.org/abs/2306.03177)

**Focus**: AEC with soft delay estimation for cases with significant echo lag.

## Status

Phases 0-4 complete, Phase 5 (GGML) has export + block-by-block C++ verification.
Model verified (7.97M params, causality OK, all gradients flow, AMP works).
Data pipeline verified with DNS5 real data (157K clean, 64K noise, 60K RIR files).
All training data packed into a single squashfs image (dns5.sqsh), mounted via
Docker entrypoint.
Evaluation script produces ERLE/PESQ/STOI/segSNR metrics, spectrograms,
delay heatmaps, and WAV audio files. GGUF export with BN folding verified
(max error 1.26e-6). All 6 C++ block tests pass (max error < 6e-6):
FE, EncoderBlock, Bottleneck (GRU+Linear), DecoderBlock (SubpixelConv2d),
CCM, AlignBlock.

Uses Docker for training (`make -C train build && make -C train train-minimal`).
Uses nix flake for C++ build (`nix develop` provides cmake + gcc).

### C++ Build Variants

The CPU build (`nix develop`) produces multiple shared libraries — one per ISA
level (SSE4.2, AVX2, AVX-512/Zen4, etc.) — selected automatically at runtime
based on CPU capabilities. All binaries and `.so` files are placed in
`ggml/build/bin/`. The backend loader searches the executable's directory, so
binaries and `.so` files must stay together.

The CUDA build (`make build-ggml` via Docker) produces binaries at
`ggml/build-cuda/` that link against shared CUDA libraries (`libcudart.so`,
`libcublas.so`) from the NGC container. These binaries must run inside the
Docker container or on a host with matching CUDA runtime.

Consumers of `libdeepvqe.so` (the C API shared library) must ensure the
`libggml-cpu-*.so` variant files are discoverable at runtime — either in the
same directory as the consumer binary or via `LD_LIBRARY_PATH`.

## Running Python Code

All Python code must run inside the Docker container (it has PyTorch, CUDA,
and all dependencies). Never run Python directly on the host.

```bash
# Build the image first (most make targets do this automatically)
make -C train build

# Run arbitrary commands via train/scripts/docker-run.sh:
./train/scripts/docker-run.sh python -c 'import torch; print(torch.cuda.is_available())'

# Existing convenience targets:
make -C train test              # Smoke test (dummy data, 2 epochs)
make -C train overfit           # Overfit test (8 tonal examples, FP32, 500 epochs)
make -C train train-minimal     # Train on DNS5 minimal subset
make -C train test-model        # Run model unit tests
make -C train test-blocks       # Run block-level verification tests
make -C train check             # Check training progress
```

`train/scripts/docker-run.sh` sets up all the standard Docker mounts (project,
checkpoints, logs, datasets, eval output, torch inductor cache) and GPU
access. It can be configured via environment variables:

| Variable | Default | Description |
|---|---|---|
| `DEEPVQE_IMAGE` | `deepvqe` | Docker image name |
| `DEEPVQE_GPU` | `all` | GPU device(s) |
| `DEEPVQE_DATA_DIR` | `./datasets_fullband` | Host data directory |
| `DEEPVQE_CKPT_DIR` | `./checkpoints` | Host checkpoint directory |
| `DEEPVQE_LOG_DIR` | `./logs` | Host log directory |
| `DEEPVQE_EVAL_DIR` | `./eval_output` | Host eval output directory |

All file paths inside the container are relative to `/workspace/deepvqe`.
Python scripts run from `/workspace/deepvqe/train`.

## Architecture

| Component | Details |
|-----------|---------|
| Sample rate | 16 kHz |
| STFT | 512 FFT, 256 hop, sqrt-Hann window, 257 freq bins |
| Mic encoder | 5 blocks: 2->64->128->128->128->128 channels |
| Far-end encoder | 2 blocks: 2->32->128 channels |
| AlignBlock | Cross-attention soft delay, dmax=32 (320ms), h=32 similarity channels |
| Encoder block 3 | 256->128 (concatenated mic + aligned far-end) |
| Bottleneck | GRU(1152->576) + Linear(576->1152) |
| Decoder | 5 blocks with sub-pixel conv + BN: 128->128->128->64->64 |
| Mask head | 1x1 Conv2d(64->27), no BN |
| CCM | 27ch -> 3x3 complex convolving mask (real-valued arithmetic) |
| Parameters | ~8.0M (full model) |

## Hardware

- Training: RTX 5070 16GB, ~2GB VRAM estimated for B=8 dmax=32 T=188 with AMP
- Datasets: ICASSP 2022 AEC + DNS challenge data

## Project Structure

```
Makefile                            # GGML build targets + training delegation
flake.nix                           # Nix dev env (cmake, gcc for C++ build)
LICENSE                             # Apache 2.0

ggml/
  CMakeLists.txt                    # CMake build with ggml submodule (CPU: dynamic ISA variants, CUDA: static)
  deepvqe.cpp                      # CLI inference binary
  deepvqe_model.cpp / .h           # Model struct + forward pass
  deepvqe_api.cpp / .h             # C API shared library
  common.h / common.cpp            # .npy I/O, comparison utilities
  test_fe.cpp                      # FE block test
  test_encoder.cpp                 # EncoderBlock test
  test_bottleneck.cpp              # Bottleneck (GRU+Linear) test
  test_decoder.cpp                 # DecoderBlock test
  test_ccm.cpp                     # CCM test
  test_align.cpp                   # AlignBlock test
  example_purego_test.go           # Go integration test
  vendor/ggml/                     # ggml library (git submodule)

train/
  Makefile                          # All Docker/training targets
  Dockerfile                        # NGC PyTorch + squashfuse
  pyproject.toml                    # uv project config
  train.py                          # Training script
  eval.py                           # Evaluation script
  export_ggml.py                    # Weight export with BN folding
  utils.py                          # Shared utilities (load_checkpoint, collate_fn, etc.)
  test_model.py                     # Model verification tests
  test_data.py                      # Data pipeline verification
  test_blocks.py                    # Block-level verification tests
  test_ccm_learning.py              # CCM learning isolation tests
  configs/
    default.yaml                    # Training configuration
    overfit.yaml                    # Overfit test configuration
  src/
    model.py                        # DeepVQEAEC (full model)
    blocks.py                       # FE, ResidualBlock, EncoderBlock, etc.
    align.py                        # AlignBlock (soft delay estimation)
    ccm.py                          # Complex convolving mask (real-valued)
    losses.py                       # Loss functions
    metrics.py                      # ERLE, PESQ, STOI
    stft.py                         # STFT/iSTFT helpers
    viz.py                          # Visualization helpers
    config.py                       # Config dataclass loading
  data/
    dataset.py                      # AECDataset class
    synth.py                        # Online audio synthesis
  scripts/
    docker-run.sh                   # Docker run wrapper
    entrypoint.sh                   # Docker entrypoint (mounts .sqsh datasets)
    compare.py                      # Layer-by-layer PyTorch/GGML comparison
    listen.py                       # WAV audio export
    diagnose_decoder.py             # Decoder activation diagnostics
    check_training.py               # Training progress checker
    report_training.py              # TensorBoard report generator
    test_erle.py                    # ERLE test with real speech
    tb_utils.py                     # TensorBoard log utilities
    download_dns5_minimal.sh        # Download DNS5 subset
    download_dns5_full.sh           # Download full DNS5
    pack_squashfs.sh                # Pack datasets into squashfs
  notebooks/
    explore_training.ipynb          # Interactive pipeline exploration
  reference/
    deepvqe_xr.py                   # Xiaobin-Rong NS-only impl (real-valued CCM)
    deepvqe_xr_v1.py               # Xiaobin-Rong NS-only impl (complex CCM)
```

## Data Setup

```bash
# Download DNS5 minimal subset (~25GB download, ~50GB unpacked)
./train/scripts/download_dns5_minimal.sh datasets_fullband

# Pack all training data into a single squashfs image.
mkdir -p datasets_fullband/_dns5_staging
cp -rl datasets_fullband/clean_fullband datasets_fullband/_dns5_staging/clean
cp -rl datasets_fullband/datasets_fullband/noise_fullband datasets_fullband/_dns5_staging/noise
cp -rl datasets_fullband/datasets_fullband/impulse_responses datasets_fullband/_dns5_staging/impulse_responses

mksquashfs datasets_fullband/_dns5_staging datasets_fullband/sqsh/dns5.sqsh \
    -comp zstd -Xcompression-level 3
rm -rf datasets_fullband/_dns5_staging

# The Docker entrypoint auto-mounts .sqsh files to /data/<name>.
# dns5.sqsh -> /data/dns5/{clean,noise,impulse_responses}
```

## Training Details

| Parameter | Value |
|-----------|-------|
| Optimizer | AdamW |
| Learning rate | 3.0e-4 |
| Weight decay | 5e-7 |
| Batch size (physical) | 32 |
| Gradient accumulation | 2 (effective batch 64) |
| Epochs | 250 |
| Clip length | 3.0 seconds (~188 frames) |
| Mixed precision | Yes (AMP) |
| Gradient clipping | 5.0 |
| Scheduler | CosineAnnealingWarmRestarts (T0=200, Tmult=1) |

## Loss Function

Multi-component (paper does not disclose, this is our design):
1. **Power-law compressed MSE** (weight=1.0): Compress pred/target STFT with c=0.3, MSE
2. **Magnitude L1** (weight=0.5): Direct magnitude accuracy
3. **Time-domain L1** (weight=0.5): Waveform reconstruction
4. **Delay cross-entropy** (weight=1.0): Supervises AlignBlock attention with ground truth delay
5. **Entropy regularization** (weight=0.01): Sharpens attention distribution

## References

- [DeepVQE paper](https://arxiv.org/abs/2306.03177) (Indenbom et al., 2023)
- [Xiaobin-Rong implementation](https://github.com/Xiaobin-Rong/deepvqe) (NS-only, clean code)
- [Okrio implementation](https://github.com/Okrio/deepvqe) (AEC path, reference)

### Reference Code (`train/reference/`)

Local copies of third-party implementations used as starting points:

- **`deepvqe_xr.py`** -- Xiaobin-Rong's NS-only DeepVQE with **real-valued CCM** (no `torch.complex`). This is the variant our CCM is based on for GGML compatibility.
- **`deepvqe_xr_v1.py`** -- Same architecture but uses **complex-valued CCM** (`torch.complex`). Kept for comparison.

#### Known Bugs in Reference Code

- **Xiaobin-Rong AlignBlock** (line 57): uses `K.shape[1]` (hidden=32) instead of `x_ref.shape[1]` (in_channels=128) for weighted sum reshape -- fixed in our `train/src/align.py`.
- **Okrio AlignBlock**: `torch.zeros()` without `.to(device)` -- fails on GPU. Fixed in our implementation.

---
> Source: [richiejp/deepvqe-ggml](https://github.com/richiejp/deepvqe-ggml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
