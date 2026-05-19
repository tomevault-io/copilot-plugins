## raglight

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
make install          # uv pip install -r pyproject.toml

# Run all tests
make test             # PYTHONPATH=src python3 -m unittest -v

# Run a single test module
PYTHONPATH=src python3 -m unittest tests.tests_rag.test_rag_pipeline

# Format code
uv run black .
```

Python >= 3.12 required. Uses `uv` as package manager.

## Architecture

RAGLight is a modular RAG library built around the **Builder pattern** for pipeline composition. The core abstraction is a LangGraph `StateGraph` (`retrieve → generate`) that orchestrates embeddings, vector store, and LLM.

### Data flow

```
FolderSource / GitHubSource
    → DocumentProcessorFactory (PDF / Code / Text / VLM-PDF)
    → EmbeddingsModel.embed_documents()
    → VectorStore.ingest()

Query → VectorStore.similarity_search() → [CrossEncoder rerank] → LLM.generate() → Answer
```

### Key abstractions (all use ABC + strategy pattern)

| Abstraction | Location | Implementations |
|---|---|---|
| `LLM` | `src/raglight/llm/llm.py` | Ollama, LMStudio, Mistral, OpenAI, Gemini |
| `EmbeddingsModel` | `src/raglight/embeddings/embeddings_model.py` | HuggingFace, Ollama, OpenAI, Gemini |
| `VectorStore` | `src/raglight/vectorstore/vector_store.py` | ChromaVS (`raglight[chroma]`), QdrantVS (`raglight[qdrant]`) |
| `DocumentProcessor` | `src/raglight/document_processing/document_processor.py` | PDF, Code, Text, VLM-PDF |

### Extending the library

- **New LLM**: extend `LLM`, implement `load()` + `generate(input: Dict) -> str`, register in `builder.py` `with_llm()`
- **New embeddings**: extend `EmbeddingsModel`, implement `load()` + `embed_documents()` + `embed_query()`, register in `builder.py` `with_embeddings()`
- **New vector store**: extend `VectorStore`, implement abstract methods, register in `builder.py` `with_vector_store()`
- **New document processor**: extend `DocumentProcessor`, implement `process()`, register in `DocumentProcessorFactory.get_processor()`

### Pipeline entry points

- `src/raglight/rag/simple_rag_api.py` — `RAGPipeline` (high-level, recommended for users)
- `src/raglight/rag/builder.py` — `Builder` (fluent API for custom pipelines)
- `src/raglight/rag/rag.py` — `RAG` (core LangGraph state machine)
- `src/raglight/rag/simple_agentic_rag_api.py` — `AgenticRAGPipeline` (agent mode with MCP tools)

### Vector store extras

Vector store backends are optional dependencies — only install what you need:

| Constant | Extra | Notes |
|---|---|---|
| `Settings.CHROMA` | `raglight[chroma]` | Requires C++ compiler on Windows |
| `Settings.QDRANT` | `raglight[qdrant]` | Pure Python, Windows-friendly |

`ChromaVS` and `QdrantVS` are lazy-loaded in `src/raglight/__init__.py` — importing `raglight` never fails even if neither extra is installed.

The env var `RAGLIGHT_DB` (default: `Chroma`) controls which backend is used by the CLI (`raglight chat`, `raglight agentic-chat`) and the REST API (`raglight serve`). `RAGLIGHT_DB_HOST` / `RAGLIGHT_DB_PORT` configure remote mode.

### Configuration

All configs are `@dataclass` in `src/raglight/config/`. Provider constants (e.g. `Settings.OLLAMA`, `Settings.CHROMA`, `Settings.QDRANT`) are defined in `src/raglight/config/settings.py`.

## Testing conventions

- Framework: `unittest` with `unittest.mock`
- Tests live in `tests/` mirroring the `src/raglight/` structure
- Shared test constants in `tests/test_config.py`
- Mock pattern: instantiate the class with `preload_model=False` (or equivalent), then inject `MagicMock()` into the relevant attribute before calling methods

---
> Source: [Bessouat40/RAGLight](https://github.com/Bessouat40/RAGLight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
