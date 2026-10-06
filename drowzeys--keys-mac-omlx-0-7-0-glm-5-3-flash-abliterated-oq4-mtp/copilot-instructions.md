## keys-mac-omlx-0-7-0-glm-5-3-flash-abliterated-oq4-mtp

> **Repo:** https://github.com/drowzeys/keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP

# Agent one-shot — Mac GLM-5.3-Flash Abliterated oQ4-MTP on oMLX 0.7.0

**Repo:** https://github.com/drowzeys/keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP  
**Weights (gated, auto-approve):** https://huggingface.co/drowzeys/keys-Mac-oMLX-0.6.3.2RC-Dual-ANE-GLM-5.3-Flash-Abliterated-oQ4  
**Engine:** oMLX **0.7.0** release wheel (sha256 pinned in `oneshot-setup.sh`), own venv, GPU + Lightning MTP fixed depth 2

```bash
git clone https://github.com/drowzeys/keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP.git
cd keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP
brew install python@3.11 uv
hf auth login   # after agreeing on the gated model card
bash oneshot-setup.sh          # installs, downloads ~175 GB, configures, serves on :11500
```

Serve id: `glm53-flash-oq4-mtp-ablit`. 256 GB Mac required. Thinking **off**. Leave context at the
engine default (1M declared; the ladder here runs to 256k).

Rules:

- Install oMLX from the **release wheel**. A `pip install git+...@v0.7.0` build has no compiled
  `glm_moe_dsa` / `decode_fast` kernels and silently runs the slow path. `oneshot-setup.sh --verify`
  checks for them.
- Keep the **config overlay** (`scripts/make-config-overlay.sh`). Vontra's config lists 46
  `mlp_layer_types` for 45 layers and oMLX 0.7.0 (transformers 5.17) refuses to load it.
- Keep **`mtp_fixed_depth: 2`**. Adaptive depth (default, max 3) and max 4 measured lower on prose.
- Do **not** enable DFlash for this checkpoint: 0.7.0's DFlash loader fails on the oQ4 mixed-bit
  `forget_gate` and silently falls back to plain decode (58.6 tok/s).
- Do not enable the 0.6.3.2RC Dual-ANE patches here. On M5 Ultra the GPU prefill path is faster.
- Restart with `scripts/serve.sh`, which frees the port first. Do not background `cd && omlx serve &`.

**Credits:** keep CREDITS.md in sync. Name the original authors (Z.ai, Vontra, dealignai / Jordan
Schenck, jundot and the oMLX 0.7.0 contributors, Blaizzy, PipeNetwork, MLX). Authors only — no credit
hyperlinks.

---
> Source: [drowzeys/keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP](https://github.com/drowzeys/keys-Mac-oMLX-0.7.0-GLM-5.3-Flash-Abliterated-oQ4-MTP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
