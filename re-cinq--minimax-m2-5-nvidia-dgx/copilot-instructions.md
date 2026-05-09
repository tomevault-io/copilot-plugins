## minimax-m2-5-nvidia-dgx

> Local inference server for MiniMax M2.5 on NVIDIA DGX Spark. OpenAI-compatible API at `localhost:8080/v1`.

# CLAUDE.md

## Project Overview

Local inference server for MiniMax M2.5 on NVIDIA DGX Spark. OpenAI-compatible API at `localhost:8080/v1`.

**Target Hardware**: NVIDIA DGX Spark (GB10 Grace Blackwell, 128GB unified memory)
**Model**: MiniMax M2.5 UD-Q3_K_XL (~101GB GGUF, 230B total params, 10B active via MoE)

## Commands

```bash
# From docker/ directory
docker compose build    # Build image (first time only)
docker compose up -d    # Start
docker compose down     # Stop
docker compose ps       # Status
docker compose logs -f  # Logs
```

### Model Download

```bash
huggingface-cli download unsloth/MiniMax-M2.5-GGUF \
  --local-dir ./models --include '*UD-Q3_K_XL*'
```

### API Test

```bash
curl http://localhost:8080/health
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "minimax-m2.5", "messages": [{"role": "user", "content": "Hello"}]}'
```

## Architecture

```
├── docker/
│   ├── docker-compose.yml       # Server config
│   └── Dockerfile.llama-server  # Build for DGX Spark (CUDA 13.0, sm_121)
├── models/                      # GGUF files (gitignored)
├── config/                      # Templates
└── tests/
```

### Inference Stack

- **Backend**: llama.cpp via Docker (local build)
- **API**: Port 8080, OpenAI-compatible `/v1`
- **Key Flags**: `-ngl 999`, `-fa on`, `-c 131072`
- **Sampling**: `--temp 1.0 --top-p 0.95 --top-k 40 --min-p 0.01` (MiniMax recommended)

## Code Style

- **Python**: ruff, type hints, Python 3.11+
- **Bash**: Google Shell Style Guide, shellcheck

## Important

Don't poll for server status in loops. Model loading takes ~5 minutes. Use:
- `docker compose ps` - container status
- `docker compose logs` - check logs
- Let user confirm when ready

---
> Source: [re-cinq/minimax-m2.5-nvidia-dgx](https://github.com/re-cinq/minimax-m2.5-nvidia-dgx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
