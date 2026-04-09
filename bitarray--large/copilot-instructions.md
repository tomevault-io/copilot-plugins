## large

> > **L**ightweight **A**rchitecture for **R**unning **G**enerative **E**ngines

# LARGE — LLM Inference in Rust

> **L**ightweight **A**rchitecture for **R**unning **G**enerative **E**ngines

An educational, from-scratch LLM inference engine written in Rust.
The goal is CPU inference on Qwen3-0.6B using the GGUF model format.

## Quick Reference

```bash
cargo build --release        # Build optimized binary
cargo run --release           # Run inference
cargo test                    # Run all tests
cargo clippy                  # Lint
cargo doc --open              # Generate & view documentation
```

## Project Structure

```
large/
├── README.md              # Project overview and quick start
├── CLAUDE.md              # This file — project conventions & architecture notes
├── Cargo.toml             # Rust package manifest (memmap2, byteorder, regex)
├── models/                # Model files (git-ignored, downloaded separately)
│   └── Qwen3-0.6B-Q8_0.gguf
├── src/
│   ├── main.rs            # Entry point — CLI, prefill, generation loop
│   ├── lib.rs             # Module declarations
│   ├── gguf.rs            # GGUF file parser (header, metadata, tensor index, mmap)
│   ├── tensor.rs          # f16/Q8_0 dequantization, mat-vec, RMSNorm, RoPE, softmax
│   ├── tokenizer.rs       # GPT-2 byte-level BPE tokenizer
│   ├── model.rs           # Qwen3 transformer (GQA, QK-Norm, SwiGLU, KV cache)
│   └── sampler.rs         # Temperature + top-p sampling
└── docs/
    ├── architecture.md    # Inference pipeline walkthrough
    ├── tokenizer.md       # How the BPE tokenizer works
    └── quantization.md    # GGUF format and Q8_0 quantization
```

## Target Model: Qwen3-0.6B

We use the **Q8_0 GGUF** quantization (639 MB) from the official Hugging Face repo:
<https://huggingface.co/Qwen/Qwen3-0.6B-GGUF>

### Architecture Parameters (from config.json)

| Parameter               | Value            | Notes                                              |
|-------------------------|------------------|----------------------------------------------------|
| Architecture            | Qwen3ForCausalLM | Decoder-only transformer                           |
| Vocabulary size         | 151,936          | Token embedding table size                         |
| Hidden size             | 1,024            | Dimension of hidden states (d_model)               |
| Intermediate size       | 3,072            | FFN inner dimension (SwiGLU, so effective = 2/3)   |
| Number of layers        | 28               | Transformer decoder blocks                         |
| Attention heads (Q)     | 16               | Query heads per layer                              |
| KV heads                | 8                | Key/Value heads (Grouped Query Attention, ratio 2:1)|
| Head dimension          | 128              | Per-head dimension (hidden_size / num_heads)       |
| Activation function     | SiLU (Swish)     | Used in the feed-forward network                   |
| RMSNorm epsilon         | 1e-6             | Layer normalization constant                       |
| RoPE theta              | 1,000,000        | Rotary Position Embedding base frequency           |
| Max position embeddings | 40,960           | Maximum sequence length the model supports         |
| Tied embeddings         | Yes              | Input & output embedding matrices are shared       |
| BOS token ID            | 151,643          | Beginning-of-sequence token                        |
| EOS token ID            | 151,645          | End-of-sequence token                              |

### Transformer Block Structure

Each of the 28 layers follows this pattern:

```
Input
  │
  ├─► RMSNorm ─► Multi-Head Attention (GQA) ─► + Residual
  │                                              │
  └──────────────────────────────────────────────┘
  │
  ├─► RMSNorm ─► SwiGLU Feed-Forward Network ─► + Residual
  │                                               │
  └───────────────────────────────────────────────┘
  │
Output
```

**Grouped Query Attention (GQA):** 16 query heads share 8 KV heads (2:1 ratio).
Each pair of query heads shares one KV head, reducing KV-cache memory by 50%
compared to standard multi-head attention.

**SwiGLU FFN:** The feed-forward block uses gated linear units with SiLU activation:
```
FFN(x) = (SiLU(x · W_gate) ⊙ (x · W_up)) · W_down
```
Where W_gate and W_up are [1024 → 3072] and W_down is [3072 → 1024].

**RoPE (Rotary Position Embeddings):** Applied to Q and K projections with
base frequency θ = 1,000,000. Encodes position information directly into
the attention computation without separate position embeddings.

## GGUF Format

GGUF (GPT-Generated Unified Format) is a binary format for storing LLM weights
and metadata in a single file. Key properties:

- **Self-contained:** Weights, tokenizer, and all config in one file
- **Memory-mappable:** Can mmap the file for efficient access
- **Quantization-aware:** Supports various quantization schemes (Q4_0, Q8_0, F16, etc.)

### Q8_0 Quantization

Q8_0 stores each group of 32 weights as:
- 1 × f16 scale factor (2 bytes)
- 32 × int8 quantized values (32 bytes)
- Total: 34 bytes per 32 weights ≈ 8.5 bits per weight

To dequantize: `weight = scale * quantized_value`

## Coding Conventions

- **Documentation:** Every public function, struct, and module must have doc comments
  explaining *what* it does and *why*. This is an educational project — explain liberally.
- **Error handling:** Use `Result<T, E>` with descriptive error types. No `.unwrap()` in
  library code (ok in tests and examples).
- **Naming:** Follow standard Rust conventions (snake_case for functions/variables,
  PascalCase for types, SCREAMING_SNAKE_CASE for constants).
- **Testing:** Each module should have a `#[cfg(test)] mod tests` section.
- **Performance comments:** When making a performance-related choice, document *why*
  with a comment (e.g., "// Using mmap here to avoid copying 639MB into heap memory").
- **No unsafe unless necessary:** Prefer safe Rust. If `unsafe` is required (e.g., for
  SIMD or mmap), isolate it and document the safety invariants.
- **Dependencies:** Minimize external crates. The educational value comes from
  implementing things ourselves. Allowed exceptions:
  - `memmap2` — for memory-mapped file I/O
  - `byteorder` — for portable binary reading
  - `serde` / `serde_json` — if needed for config parsing
  - Other crates only with explicit justification

## Inference Pipeline Overview

The high-level inference loop we'll implement:

```
1. Load GGUF file (parse header, metadata, tensor index)
2. Memory-map the weight data
3. Build tokenizer from GGUF metadata
4. Tokenize the input prompt
5. Forward pass through the model:
   a. Token embedding lookup
   b. For each of 28 transformer layers:
      i.   RMSNorm
      ii.  GQA Self-Attention (with RoPE)
      iii. Residual connection
      iv.  RMSNorm
      v.   SwiGLU FFN
      vi.  Residual connection
   c. Final RMSNorm
   d. Language model head (project to vocab logits)
6. Sample next token from logits (temperature, top-p, etc.)
7. Repeat from step 5 until EOS or max length
```

## Resources

- [Qwen3 Model Card](https://huggingface.co/Qwen/Qwen3-0.6B)
- [Qwen3 Technical Report](https://arxiv.org/abs/2505.09388)
- [GGUF Specification](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)
- [llama.cpp](https://github.com/ggerganov/llama.cpp) — reference C++ implementation
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bitarray)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bitarray)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
