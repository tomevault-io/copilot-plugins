## virtual-contributor-ingest-website

> Website ingestion service for the Alkemio platform. Receives ingestion requests via RabbitMQ, recursively crawls a target website, extracts text content from HTML pages, splits large documents into chunks, generates per-document and body-of-knowledge summaries using Mistral AI via LangGraph summarize-then-refine workflows, embeds all content into ChromaDB via the shared engine library, and publishes results back to RabbitMQ.

# virtual-contributor-ingest-website Development Guidelines

## Project Overview

Website ingestion service for the Alkemio platform. Receives ingestion requests via RabbitMQ, recursively crawls a target website, extracts text content from HTML pages, splits large documents into chunks, generates per-document and body-of-knowledge summaries using Mistral AI via LangGraph summarize-then-refine workflows, embeds all content into ChromaDB via the shared engine library, and publishes results back to RabbitMQ.

## Active Technologies

- Python 3.12+
- alkemio-virtual-contributor-engine v0.8.0 (base library: `mistral_small` LLM, `ingest_documents`, `setup_logger`, RabbitMQ consumer, shared types)
- LangGraph (summarization state machines: `document_graph`, `bok_graph`)
- LangChain + langchain-text-splitters (prompt templates, recursive text splitting)
- BeautifulSoup4 + requests (web crawling and HTML parsing)
- ChromaDB (vector database, managed via engine library)
- Mistral AI mistral-small (LLM provider for summarization)
- Scaleway-hosted Qwen3-Embedding-8B (embeddings provider)
- aio-pika (RabbitMQ async client, via engine library)
- LangSmith (LLM tracing)

## Project Structure

```text
.
├── main.py              # Entry point: RabbitMQ consumer, crawling (get_pages),
│                        # document extraction (get_documents), preparation
│                        # (prepare_documents), embedding (embed_documents)
├── graph.py             # LangGraph summarization: document_graph, bok_graph,
│                        # progressive length budget, prompt templates
├── config.py            # Env dataclass (environment variable loading)
├── url_utils.py         # is_file_link URL classification utility
├── local_types.py       # DocumentType enum
├── pyproject.toml       # Dependencies (Poetry)
├── Dockerfile           # Multi-stage build (builder, runtime, runtime-full)
├── .env.default         # Environment variable documentation
├── .flake8              # Linting configuration
├── tests/               # pytest test suite
│   ├── conftest.py      # Shared fixtures and mocks
│   ├── test_main.py     # Tests for crawling, documents, preparation, embedding
│   ├── test_graph.py    # Tests for graph builder, progressive length, nodes
│   ├── test_config.py   # Tests for Env dataclass
│   ├── test_url_utils.py    # Tests for is_file_link
│   └── test_local_types.py  # Tests for DocumentType enum
└── .github/workflows/   # CI/CD pipelines
```

## Commands

```bash
# Install dependencies
poetry install

# Run the service
poetry run python main.py

# Run tests
poetry run pytest

# Run tests with coverage
poetry run pytest --cov=main --cov=graph --cov=config --cov=url_utils --cov=local_types \
    --cov-report=term-missing --cov-fail-under=90

# Run linting
poetry run flake8

# Build Docker image (runtime)
docker build -f Dockerfile --target runtime -t ingest-website:dev .

# Build Docker image (full, with Git + Hugo)
docker build -f Dockerfile --target runtime-full -t ingest-website:full .
```

## Code Style

- Follow flake8 rules (max-line-length=100, see `.flake8`)
- Use `setup_logger(__name__)` for all logging — never `print()`/`pprint()`
- All async request handling via aio-pika — no sync HTTP endpoints
- External dependencies (LLM, embeddings, vector DB) accessed via the engine library
- All config via environment variables — never hardcode credentials

## Key Patterns

- **Ingestion pipeline**: `get_pages()` → `get_documents()` → `prepare_documents()` → `embed_documents()`
- **Recursive crawling**: `get_pages()` follows links within the base URL domain, skips file links, respects `PROCESS_PAGES_LIMIT`
- **Two-tier summarization**: Documents with >3 chunks get individual summaries via `document_graph`; all content aggregated into a single body-of-knowledge summary via `bok_graph`
- **LangGraph pattern**: `_build_graph(system_prompt, summarize_template, refine_template)` creates a summarize-then-refine state machine (initial_summary → should_refine → refine_summary loop)
- **Progressive length budget**: `_progressive_length()` scales summary target from 40% to 100% of `SUMMARY_LENGTH` based on chunk position — avoids front-loading
- **Embedding delegation**: `ingest_documents(collection_name, documents)` from engine library handles ChromaDB collection management, batching, and embedding generation
- **Collection naming**: `{urlparse(base_url).netloc}-knowledge` with colons replaced by hyphens

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alkem-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alkem-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
