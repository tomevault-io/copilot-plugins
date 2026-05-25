## infinity-emb-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an infinity-emb-server project that provides local embedding and reranking API services using Japanese-specialized models. The project runs two servers:

1. **Embedding server** (port 7997): Uses `cl-nagoya/ruri-v3-310m` model
2. **Reranker server** (port 7998): Uses `cl-nagoya/e5-ja-reranker-large` model

Both servers provide OpenAI-compatible API endpoints.

## Essential Commands

### Setup
```bash
# Install dependencies
uv sync
```

### Running Services
```bash
# Start both servers
./start_infinity.sh

# Stop both servers
./start_infinity.sh stop

# Run only the reranker (if needed)
./run_rerank.sh
```

### Testing
```bash
# Test embedding server
curl http://localhost:7997/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model":"cl-nagoya/ruri-v3-310m","input":["瑠璃色とは？"]}'

# Test reranker server
curl http://localhost:7998/v1/rerank \
  -H "Content-Type: application/json" \
  -d '{"model":"cl-nagoya/e5-ja-reranker-large","query":"query text","documents":["doc1","doc2"]}'
```

## Architecture

The project uses a simple shell script orchestration pattern:

- **start_infinity.sh**: Main orchestration script that manages both servers
  - Starts processes in background with PID tracking via `/tmp/infinity_{emb,rerank}.pid`
  - Logs output to `./logs/infinity-{emb,rerank}.log`
  - Configured for MPS (Apple Silicon) by default, can be changed to "cuda" or "cpu"

- **Service Configuration**:
  - Both servers use `uv run infinity_emb v2` command
  - Embedding: batch-size 128, port 7997
  - Reranker: batch-size 32, port 7998
  - Both listen on 0.0.0.0 (network accessible)

## Key Technologies

- **Python 3.12+** with **uv** package manager
- **infinity-emb[all]**: Core embedding server framework
- **PyTorch** with MPS support for Apple Silicon
- **transformers**, **sentencepiece**, **fugashi**: NLP dependencies for Japanese text processing

## Important Notes

- The models are Japanese-specialized and downloaded automatically on first run
- Logs are stored in `./logs/` directory
- The project is optimized for macOS with MPS (Metal Performance Shaders)
- Both servers expose OpenAI-compatible endpoints under `/v1` prefix

---
> Source: [ylabo0717/infinity-emb-server](https://github.com/ylabo0717/infinity-emb-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
