## llm-config

> LLM configuration and training defaults for this project

# LLM Configuration Defaults

- Tokenizer: SentencePiece BPE, vocab=1024; special tokens `[PAD],[BOS],[EOS],[UNK]`
- Context length: 256
- Model: GPT-style, d_model=128, n_layers=7, n_heads=4, GELU, pre-norm, dropout≤0.1, tied LM head
- Optimizer: AdamW; lr=3e-4; warmup=200; cosine decay; wd=0.1; grad clip=1.0
- Batch tokens target: ~8k per optimization step (dynamic batching/accumulation)
- Eval interval: 200 steps; checkpoint `best` and `last`
- Hardware: Apple Silicon; MLX backend

---
> Source: [saivishnu2299/LocalMacLLM](https://github.com/saivishnu2299/LocalMacLLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
