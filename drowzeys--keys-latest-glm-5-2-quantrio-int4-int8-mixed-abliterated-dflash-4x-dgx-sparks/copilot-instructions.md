## keys-latest-glm-5-2-quantrio-int4-int8-mixed-abliterated-dflash

> 1. Read `.specify/memory/constitution.md` first.

# Agent instructions (this repo only)

1. Read `.specify/memory/constitution.md` first.  
2. Execute `scripts/` in numeric order only. Never skip gates.  
3. **Weights = one HF package, two parts — both required:**  
   - Package: https://huggingface.co/drowzeys/keys-latest-GLM-5.2-Quantrio-INT4-INT8-Mixed-Abliterated-DFlash  
   - **Part A (target):** repo root — 124× `model-*-of-00124.safetensors`  
   - **Part B (draft):** [`dflash/`](https://huggingface.co/drowzeys/keys-latest-GLM-5.2-Quantrio-INT4-INT8-Mixed-Abliterated-DFlash/tree/main/dflash) — `dflash/model.safetensors`  
   - Full commands: `downloads/MANIFEST.md` · script: `scripts/02-download-weights.sh`  
   - Abort if either part is missing.  
4. Locked env: `serve/champion-128k-k12-s4.env`  
   - `MODEL=…/glm52-int4-int8mix-abliterated`  
   - `DFLASH_MODEL=…/glm52-int4-int8mix-abliterated/dflash`  
   - `KV_BYTES=10000000000` · `MAX_MODEL_LEN=128000` · `MAX_NUM_SEQS=4` · `DFLASH_K=12` · `UTIL=0.85`  
5. If launch fails with “8.87 GiB KV cache is needed”, **increase KV pin**, do not lower context silently.  
6. Do not use `keys-latest-GLM-5.2-DFlash-FP8` standalone or old C1-30 titles.  
7. UTIL hard cap 0.85 — do not “squeeze” above.  

---
> Source: [drowzeys/keys-latest-GLM-5.2-Quantrio-INT4-INT8-Mixed-Abliterated-DFlash-4x-DGX-Sparks](https://github.com/drowzeys/keys-latest-GLM-5.2-Quantrio-INT4-INT8-Mixed-Abliterated-DFlash-4x-DGX-Sparks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
