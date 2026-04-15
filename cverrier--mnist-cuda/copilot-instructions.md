## mnist-cuda

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
make all          # Build display_digits and bswap tests
make display      # Build and run digit display program
make test         # Build and run C tests (test_bswap, test_mnist)
make test-cuda    # Build and run CUDA model tests
make train        # Build and run training on MNIST
make clean        # Remove build artifacts
```

Build outputs go to `bin/` directory.

**Compilers:**
- C: Clang with C23 standard, strict warnings enabled
- CUDA: NVCC with `-O2 -g`

**CUDA setup:** Set `CUDA_PATH` environment variable if not at `/usr/local/cuda`.

## Architecture

CUDA-accelerated multilayer perceptron for MNIST digit classification.

**Network:** 784 (input) → 128 (hidden, ReLU) → 10 (output logits)

**Training pipeline:**
```
MNIST IDX files → MNISTDataset → normalize_mnist() → NormalizedMNIST
                                                           ↓
                                              copy to GPU (d_input)
                                                           ↓
                                    forward() → compute_loss() → backward() → update_params()
```

**Core modules:**
- `model.cu/h` - CUDA neural network: forward pass, fused softmax+cross-entropy loss, backprop, SGD, accuracy
- `train.cu` - Complete training program: loads MNIST, copies to GPU, runs training loop with accuracy reporting
- `mnist.c/h` - Loads IDX format files, validates headers (big-endian), normalizes pixels to float
- `display.c/h` - ASCII art visualization of digit samples
- `bswap.c/h` - 32-bit byte-swap for big-endian header parsing

**CUDA kernels (in model.cu):**
- `linear_relu_kernel` - Fused linear layer + ReLU for hidden layer
- `linear_kernel` - Linear layer for output logits
- `softmax_cross_entropy_kernel` - Fused softmax, loss, and gradient computation (numerically stable)
- `matmul_at_b_kernel` - Matrix multiply with transposed first arg: C = A^T × B (for weight gradients)
- `bias_grad_kernel` - Bias gradient via sum over batch dimension
- `hidden_grad_kernel` - Backprop through layer 2 with ReLU mask
- `sgd_kernel` - SGD parameter update: param -= lr × grad
- `count_correct_kernel` - Count correct predictions by comparing argmax(logits) to labels

**Data structs:**
- `Model` - Network weights/biases on GPU: `d_W1`, `d_b1`, `d_W2`, `d_b2` (all device pointers)
- `TrainState` - Training buffers on GPU: activations (`d_hidden`, `d_logits`), gradients, loss scalar
- `MNISTDataset` - Raw data: `uint8_t* pixels` (0-255), `uint8_t* labels` (0-9)
- `NormalizedMNIST` - Processed: `float* pixels` (0.0-1.0), shares labels pointer with source

**Memory management:**
- All `Model` and `TrainState` pointers are GPU device memory (prefix `d_`)
- MNIST labels pointer is shared between raw and normalized datasets (only free once)
- Use `CUDA_CHECK` macro for all CUDA API calls

## Data Files

MNIST data expected at:
- Training: `data/mnist/train-images.idx3-ubyte` and `data/mnist/train-labels.idx1-ubyte`
- Test: `data/mnist/t10k-images.idx3-ubyte` and `data/mnist/t10k-labels.idx1-ubyte`

## Code Style

Formatting enforced by `.clang-format` (LLVM style, 2-space indent). Pre-commit hooks run clang-format checks.

## Current Status

**Implemented:** Data loading, normalization, model creation with Xavier init, forward pass, fused softmax+cross-entropy loss with gradient computation, backward pass (gradient computation for all parameters), SGD parameter updates, accuracy computation, full training loop.

**TODO:** Explore optimizations (shared memory, tiling, cuBLAS).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cverrier)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cverrier)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
