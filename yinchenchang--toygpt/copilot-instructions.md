## toygpt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Character-level transformer language model (ToyGPT) trained on Shakespeare text, based on Andrej Karpathy's "Let's build GPT" tutorial. Modernized to PyTorch 2.x standards (2026). Implemented as Python scripts (and legacy Jupyter notebooks) using PyTorch 2.0+.

## Running

```bash
python Andrej_GPT_Test.py                    # Auto-detects GPU/CPU (recommended)
python Andrej_ToyGPT_Test_Merge_CPU.py       # CPU-only
python Andrej_ToyGPT_Test_Merge_GPU.py       # GPU-optimized
```

Requires: PyTorch 2.0+ and `input.txt` (Shakespeare corpus, ~1.1M chars) in the same directory.

## Script Variants

- **Andrej_GPT_Test.py** — Primary script with CUDA auto-detection
- **Andrej_ToyGPT_Test_Merge_CPU.py** — Forces `device='cpu'`, GPU features disabled
- **Andrej_ToyGPT_Test_Merge_GPU.py** — GPU-optimized with torch.compile, AMP, TF32

All three share the same model architecture; they differ in device targeting and GPU acceleration features.

## Architecture

Pre-norm transformer decoder stack for next-character prediction:

```
Character indices → Token Embedding + Positional Embedding
  → 8x Transformer Blocks (each: LayerNorm → CausalSelfAttention → LayerNorm → FeedForward, with residual connections)
  → LayerNorm → Linear head → logits (vocab_size=65)
```

Key classes: `GPTConfig` (dataclass), `CausalSelfAttention` (Flash Attention), `FeedForward` (GELU), `Block`, `GPTLanguageModel`.

## Default Hyperparameters

| Parameter | Value |
|-----------|-------|
| n_layer / n_head | 8 / 8 |
| n_embd | 128 |
| block_size (context length) | 128 |
| batch_size | 32 |
| max_iters | 5000 |
| learning_rate | 1e-3 → 1e-4 (cosine with warmup) |
| warmup_iters | 100 |
| weight_decay | 0.1 (excluded from biases/LayerNorm) |
| dropout | 0.0 |

Model size: ~1.6M parameters.

---
> Source: [YinchenChang/ToyGPT](https://github.com/YinchenChang/ToyGPT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
