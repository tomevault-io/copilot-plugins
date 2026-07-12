## wan-streamer-replication

> Reproducing Wan-Streamer v0.1 (arXiv 2606.25041) — an end-to-end real-time full-duplex audio-visual interaction model from Alibaba's Wan Team. Single Transformer for text/audio/video input & output with block-causal attention.

# Wan-Streamer Reproduction

Reproducing Wan-Streamer v0.1 (arXiv 2606.25041) — an end-to-end real-time full-duplex audio-visual interaction model from Alibaba's Wan Team. Single Transformer for text/audio/video input & output with block-causal attention.

## Project

- **Stack**: Python 3.10+, PyTorch 2.4+, no package manifest (`requirements.txt` only).
- **Phase**: Early skeleton — placeholder NN blocks, working attention mask + flow matching logic, proven test suite. No weights, no training scripts yet.
- **Key external deps** (not vendored): Qwen2.5 (Apache 2.0), Wan2.1/Wan-VAE (Apache 2.0), Mimi from Kyutai (CC-BY 4.0).
- **Paper**: `doc/paper.md` (full text, 19KB), `doc/paper.pdf` (4.8MB), `doc/main.tex` (LaTeX source).

## Commands

```bash
# Test (27 tests, all passing)
python -m pytest tests/ -v

# Install deps (Wan2.1 must be installed from source)
pip install -r requirements.txt
git clone https://github.com/Wan-Video/Wan2.1 && cd Wan2.1 && pip install -e .
```

No build/lint/formatter configured yet (no `pyproject.toml`, `Makefile`, or CI).

## Architecture

```
configs/model_config.py          # @dataclass configs: Default (Qwen2.5-7B), Medium (1.5B), Small (0.5B)
src/
  models/
    transformer.py               # WanStreamerTransformer — unified multimodal transformer skeleton
    attention.py                 # Block-causal attention mask (StreamBlock, create_streaming_mask, parse_layout)
    audio_vae.py                 # StreamingAudioVAE — Mimi wrapper (placeholder)
    video_vae.py                 # StreamingVideoVAE — Wan-VAE wrapper (placeholder)
    flow_matching.py             # LinearFlowScheduler, FlowMatchingLoss, FlowODESolver, FlowDistillationLoss
  data/
    tokenizers.py                # UnifiedTokenizer — text (Qwen), audio (Mimi), video (Wan-VAE)
    dataset.py                   # InteractionDataset, StreamingInteractionDataset (stubs)
  training/
    trainer.py                   # WanStreamerTrainer — three-stage (pretrain / interaction / distillation)
    losses.py                    # CombinedLoss — CE + Flow Matching MSE
  inference/
    thinker.py                   # Thinker (GPU 0) — encode, state update, decode, KV exchange
    performer.py                 # Performer (GPU 1) — flow-matching latent generation from KV context
tests/
  test_attention.py              # 10 tests: block-causal mask rules (same-block, agent→user, future blocked, past visible)
  test_flow_matching.py          # 8 tests: interpolation, velocity, Euler solver, loss monotonicity
  test_model.py                  # 9 tests: configs, ModalEmbedding, mask integration
scripts/                         # Empty — training scripts planned but not implemented
doc/
  paper.md                       # Full paper in readable Markdown
  paper.pdf                      # Official arXiv PDF
  main.tex / paper.bib           # LaTeX source + bibliography
  SPEC.md / ARCHITECTURE.md / RESEARCH.md  # Reproduction docs
```

**Key awareness**: `TransformerBlock` in `transformer.py` is a placeholder (`nn.MultiheadAttention`) — must be replaced with Qwen2.5 RoPE + GQA + FlashAttention. Block-causal attention is the novel component with no open-source prior; `attention.py` is the canonical implementation.

## Conventions

- **Docstrings**: Google-style with `Args:` / `Returns:` sections.
- **Classes**: `nn.Module` subclassing for models; `@dataclass` for configs and data containers.
- **Tests**: `pytest`; class-per-feature (`TestBlockCausalMask`), method-per-scenario (`test_mask_agent_sees_user`); explicit imports from `src.*`.
- **Type hints**: On all public function/method signatures (`List`, `Optional`, `Tuple`, `torch.Tensor`).
- **Section separators**: `# ----` (78-char line) in large files.
- **Placeholders**: Marked with `# TODO:` or `raise NotImplementedError` — many components are stubs awaiting real weight loading.
- **No formatter configured** — match surrounding style; use 4-space indent, 100-ish char lines.

## Notes

- `scripts/` is empty — the README references `train_stage1.py` etc. but they don't exist yet.
- Wan2.1 must be cloned and installed separately; it's not pip-installable.
- Mimi is bundled in `transformers>=4.45` as `MimiModel` — no extra package needed.
- FlashAttention is optional; `attention.py` uses standard PyTorch masking.
- The project targets a 2-GPU deployment (thinker + performer), but training needs 8×A100/H100 with FSDP.

---
> Source: [windward47/Wan-Streamer-Replication](https://github.com/windward47/Wan-Streamer-Replication) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
