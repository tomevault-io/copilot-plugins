## movie-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Movie RAG is a Retrieval-Augmented Generation system for movie information built with Python 3.12, Poetry, LangChain, and LangGraph. It supports multimodal queries (text + poster images), multiple retrieval strategies, and a hierarchy of agents with increasing autonomy.

## Development Commands

### Installation
- `make install` - Install base dependencies
- `make install-dev` - Install with dev tools (ruff, black, mypy, pytest)
- `make install-all` - Install all dependency groups (base + ml + dev + rag-agent)
- Dependencies are grouped in pyproject.toml: `base`, `ml`, `dev`, `rag-agent`; add new deps with `poetry add --group <group>`

### Testing and Quality
- `make test` or `pytest` - Run all tests
- `pytest path/to/test_file.py::test_function` - Run a single test
- `make format` - Run ruff format + check (not Black/flake8 — the Makefile uses ruff)

### Development Environment
- `make jupyter` - Start Jupyter Lab
- `make clean` - Clean cache and temp files

## Architecture

### Data Layer (`src/data/`)
- `sqlite_database.py` — SQLite DB with 8,000+ movies; indexed on ratings, genres, directors, actors, years
- `document_creators.py` — Converts CSV files (Rotten Tomatoes reviews, OMDB plots) into LangChain `Document` objects
- `chunk.py` — Custom sentence-level chunking logic

### Retriever Layer (`src/retrievers/`)
All retrievers inherit from `CustomBaseRetriever` (`base.py`) and share the interface: `add_documents`, `search`, `save`, `load`.
- **Dense** (`dense_retriever.py`): FAISS + Sentence-Transformers or OpenAI embeddings
- **Sparse** (`sparse_retriever.py`): BM25 keyword search
- **Hybrid** (`hybrid_retriever.py`): Weighted combination of dense + sparse (configurable `alpha`)
- **Visual** (`visual_retriever.py`): CLIP (ViT-B/32) embeddings on movie posters; fuses image + text via concatenation or weighted average
- `factory.py` — `create_text_retriever()` instantiates any retriever from a config dict

### RAG Chain Layer (`src/langchain/`)
- `chains/movie_rag.py` — `MovieRAGChain`: main pipeline (load docs → chunk → build retriever → optional HyDE/reranking → QA chain)
- `retrieval/hyde.py` — HyDE retriever: LLM generates a hypothetical answer, then searches with that
- `retrieval/reranker.py` — Cross-encoder reranking (Cohere or sentence-transformers)
- `retrieval/retrievers.py` — `TextRetrieverWrapper`: bridges custom retrievers into LangChain's retriever interface
- `chains/self_rag.py` — Self-RAG variant with self-critique loop
- `chains/multimodal.py` — Full multimodal chain combining text + visual retrievers
- `chains/streaming.py` — Streaming response support
- `prompts.py` — All system prompts (QA, planning, synthesis, reranking)
- `observability.py` — LangSmith tracing setup
- `ragas.py` — RAGAS evaluation metrics

### Agent Layer (`src/agents/`)
Four agent types with escalating capability, all built on LangGraph state machines:

| Agent | File | Use case |
|---|---|---|
| ReAct | `react.py` | Simple queries; reason-act-observe loop |
| Plan-Execute | `plan_execute.py` | Multi-step queries; plans then executes sequentially |
| Adaptive Plan-Execute | `adaptive_plan_execute.py` | Auto-retries failures, falls back to web search, replans via LLM |
| HITL Adaptive | `hitl_adaptive_plan_execute.py` | All adaptive features + interactive human intervention when stuck |

All agents inherit from `base_movie_agent.py` (`MovieAgent`) and compose the same tool set:
- **Text RAG** — searches plots + reviews
- **Visual RAG** — CLIP poster search
- **Combined RAG** — text + visual fusion
- **SQL** (`tools/sql_tool.py`) — LLM-generated SQL on the metadata DB (ratings, counts, filters)
- **Collaborative Filtering** (`tools/collaborative_filtering.py`) — item-based CF via critic rating cosine similarity
- **Web Search** (`tools/web_search.py`) — Tavily API; used as fallback in adaptive agents

### Notebooks (`notebooks/`)
Numbered 1–12 showing progressive feature development: embeddings → chunking → text retrieval → BM25 → visual retrieval → LangChain RAG → reranking → HyDE/streaming/LangSmith → self-RAG/RAGAS → multimodal chain → ReAct agent → plan-execute agent. `notebooks/data_prep/` contains one-time data preparation scripts.

## Required Environment Variables
```
OPENAI_API_KEY       # Embeddings + LLM
OMDB_API_KEY         # Plot and poster data
TAVILY_API_KEY       # Web search fallback (adaptive agents)
LANGCHAIN_API_KEY    # Optional: LangSmith tracing
LANGCHAIN_TRACING_V2 # Optional: enable LangSmith
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sirandou)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sirandou)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
