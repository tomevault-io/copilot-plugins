## dream-of-the-red-chamber-agent

> This is a Chinese-language RAG (Retrieval-Augmented Generation) assistant for "Dream of the Red Chamber" (《红楼梦》). It uses hybrid search (BM25 sparse + dense vector) in Milvus, neighbor chunk expansion, and LLM-powered Q&A via a Gradio web UI.

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a Chinese-language RAG (Retrieval-Augmented Generation) assistant for "Dream of the Red Chamber" (《红楼梦》). It uses hybrid search (BM25 sparse + dense vector) in Milvus, neighbor chunk expansion, and LLM-powered Q&A via a Gradio web UI.

### Services

| Service | Description | How to start |
|---------|-------------|--------------|
| **Milvus** (vector DB) | Core dependency for hybrid search. Requires Docker. | `docker compose -f /tmp/milvus/docker-compose.yml up -d` then verify with `curl http://localhost:9091/healthz` |
| **Gradio app** | Web UI on port 7860 | `cd /workspace/scripts && PYTHONPATH=/workspace:/workspace/scripts TOKENIZERS_PARALLELISM=false python3 home_page.py` |

### Important caveats

- **Mixed imports**: The codebase uses both relative imports (`from retrieve import ...`) and absolute imports (`from scripts.all_llm import ...`). You must set `PYTHONPATH=/workspace:/workspace/scripts` when running any script.
- **Working directory**: Always `cd /workspace/scripts` before running the app scripts (`home_page.py`, `run_agent.py`).
- **API keys**: A `.env` file in `/workspace/` is loaded by `python-dotenv`. Required: `DEEPSEEK_API_KEY`, `DEEPSEEK_BASE_URL`. Optional: `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `TAVILY_API_KEY`. Without `TAVILY_API_KEY` set (even to a placeholder), the import in `env_utils.py` will fail with `TypeError`.
- **Milvus data ingestion**: The `step4_ingest_hybrid.py` script must be run from `/workspace/scripts` (not from `setup_docs/`) so the relative path `../data_clean/chunks.jsonl` resolves correctly. Ingestion takes ~10 minutes on CPU as it encodes 1132 chunks with `BAAI/bge-large-zh-v1.5`.
- **No requirements.txt**: Dependencies are not formally declared. Key packages: `pymilvus`, `sentence-transformers`, `numpy`, `gradio`, `langchain-openai`, `langchain-community`, `langchain-classic`, `python-dotenv`.
- **No lint/test config**: The repository has no linter configuration, test framework, or build system.
- **Docker for Milvus**: Milvus standalone runs via Docker Compose with etcd + MinIO. The compose file is at `/tmp/milvus/docker-compose.yml`. In the Cloud Agent DinD environment, you need `fuse-overlayfs` storage driver and `iptables-legacy`.

---
> Source: [lisz1012/dream_of_the_red_chamber_agent](https://github.com/lisz1012/dream_of_the_red_chamber_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
