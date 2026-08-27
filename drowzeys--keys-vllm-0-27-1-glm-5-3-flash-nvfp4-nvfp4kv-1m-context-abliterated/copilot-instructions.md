## keys-vllm-0-27-1-glm-5-3-flash-nvfp4-nvfp4kv-1m-context-ablitera

> **Repo:** https://github.com/drowzeys/keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated

# Agent one-shot — GLM-5.3-Flash NVFP4 ablit · 1M ctx · 2× DGX Spark

**Repo:** https://github.com/drowzeys/keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated  
**Weights (gated, auto-approve):** https://huggingface.co/drowzeys/keys-GLM-5.3-Flash-NVFP4-ablit-l15-45-anchorstock  
**Runtime image (GHCR):** `ghcr.io/drowzeys/keys-vllm-glm53-flash-nvfp4-ablit:b12x-cu130`  
(`sha256:f722ec19d8260833e948d5bf46949d9ac574841860060caa24213cf550d1a41b`)

Day-0 base is `vllm/vllm-openai:glm53-flash-arm64-cu130` plus `b12x==1.2.6` / cutlass-dsl 4.7.0 (`--no-deps`, NCCL 2.30.7 kept).

```bash
git clone https://github.com/drowzeys/keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated.git
cd keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated
bash oneshot.sh                          # pull GHCR image + weights + topk2044 sibling
# edit launcher/launch.sh node IPs / NFS paths, then on the head:
ENGINE=official CTX=1048576 BLOCK_SIZE=7168 BTOK=4096 MAX_SEQS=2 SPEC=nextn MTP_K=2 UTIL=0.85 ./launcher/launch.sh
```

**Hard cap:** `--gpu-memory-utilization` / `UTIL` **≤ 0.85**. Do not raise it to fit 1M; drop `max-num-seqs` or `max-num-batched-tokens` instead.

Overlays in this repo bind-mount onto the image (zero-RoPE shim, B12X NVFP4 KV). Do not smash MTP. Spark `.1`/5482 is off-limits (Inkling).

---
> Source: [drowzeys/keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated](https://github.com/drowzeys/keys-vLLm.0.27.1-GLM-5.3-Flash-NVFP4-NVFP4KV-1M-Context-Abliterated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
