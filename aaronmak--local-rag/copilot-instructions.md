## local-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A local RAG (Retrieval-Augmented Generation) system built with Python, using Ollama for local LLM inference, ChromaDB for vector storage, and LangChain for orchestration. The system enables question-answering over custom documents entirely on the local machine.

## Technology Stack

- **Language**: Python 3.9+
- **Package Manager**: uv (fast Python package installer)
- **LLM**: Ollama (local inference)
- **Vector Database**: ChromaDB
- **Framework**: LangChain
- **Document Processing**: pdfplumber for layout-aware PDF extraction, pypdf for basic extraction
- **Testing**: pytest with coverage
- **Code Quality**: Black (formatter), Ruff (linter), mypy (type checker)

## Common Commands

### Setup and Installation

```bash
# Install dependencies
uv sync

# Install with dev dependencies
uv sync --extra dev

# Install Ollama models (external requirement)
ollama pull llama3:8b
ollama pull nomic-embed-text
```

### Running the Application

```bash
# Start interactive chat interface
uv run python scripts/chat.py

# Ingest documents from a directory (supports .txt and .pdf files)
# By default, PDFs are processed with layout preservation (bounding boxes, position, fonts)
uv run python scripts/ingest_documents.py data/documents/

# Ingest with vector store reset
uv run python scripts/ingest_documents.py data/documents/ --reset

# Ingest without layout preservation (faster, but loses spatial context)
uv run python scripts/ingest_documents.py data/documents/ --no-layout
```

### Development Commands

```bash
# Run all tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=src/local_rag --cov-report=html

# Run a single test file
uv run pytest tests/test_pipeline.py

# Run a specific test
uv run pytest tests/test_pipeline.py::test_pipeline_initialization

# Format code
uv run black src/ tests/ scripts/

# Lint code
uv run ruff check src/ tests/ scripts/

# Type check
uv run mypy src/
```

## Code Architecture

### Component Structure

The RAG system is organized into modular components:

1. **config.py**: Centralized configuration using Pydantic settings
   - Loads from environment variables or `.env` file
   - Type-safe configuration with validation
   - Settings for Ollama, ChromaDB, and RAG parameters

2. **embeddings.py**: Embedding management
   - Wraps Ollama embeddings via LangChain
   - Handles both document and query embeddings
   - Uses `nomic-embed-text` model by default

3. **vectorstore.py**: Vector storage and retrieval
   - ChromaDB integration with persistent storage
   - Document chunking using RecursiveCharacterTextSplitter
   - Similarity search with optional scoring
   - Collection management (add, delete, count)

4. **generator.py**: Text generation
   - Ollama LLM integration for response generation
   - Customizable prompt templates
   - Streaming support for real-time responses
   - Context-aware generation using retrieved documents

5. **pipeline.py**: Main orchestrator
   - High-level API combining all components
   - Methods: `add_documents()`, `query()`, `query_stream()`, `get_stats()`, `reset()`
   - Manages component lifecycle and data flow

### Data Flow

```
User Query
    ↓
[Embedding Manager] - Convert query to vector
    ↓
[Vector Store] - Similarity search for relevant documents
    ↓
[Generator] - Generate response using LLM + context
    ↓
Response to User
```

### Configuration System

The project uses a layered configuration approach:

- Default values in `config.py`
- Override via `.env` file (copy from `.env.example`)
- Override via environment variables
- Override programmatically via `Settings` object

Key settings:

- `OLLAMA_MODEL`: LLM for generation (default: llama3:8b)
- `OLLAMA_EMBEDDING_MODEL`: Model for embeddings (default: nomic-embed-text)
- `CHUNK_SIZE`: Text chunk size (default: 1000)
- `TOP_K`: Number of documents to retrieve (default: 4)

### Testing Strategy

Tests are located in `tests/` and use pytest:

- Fixtures for test settings and pipeline initialization
- Cleanup after each test to avoid state pollution
- Test ChromaDB uses separate collection (`test_documents`)
- Coverage tracking with pytest-cov

## Project Structure

```
src/local_rag/          # Main package
├── __init__.py         # Exports RAGPipeline
├── config.py           # Settings management
├── embeddings.py       # Ollama embeddings wrapper
├── vectorstore.py      # ChromaDB integration
├── generator.py        # Ollama LLM generation
├── pipeline.py         # Main RAG orchestrator
└── pdf_processor.py    # PDF layout-aware processing

tests/                  # Test suite
├── __init__.py
└── test_pipeline.py    # Pipeline integration tests

scripts/                # Utility scripts
├── chat.py             # Interactive terminal chat interface
└── ingest_documents.py # Document ingestion tool

data/documents/         # Place documents to ingest here
```

## Development Notes

### Adding New Features

When extending the system:

- Keep components modular and single-purpose
- Update configuration in `config.py` if new settings are needed
- Add tests in `tests/` for new functionality
- Update example scripts if the API changes
- Follow the existing patterns for error handling

### Working with Ollama

Ensure Ollama is running locally:

```bash
ollama serve  # If not running as service
```

List available models:

```bash
ollama list
```

Change models by updating `.env`:

```env
OLLAMA_MODEL=mistral
OLLAMA_EMBEDDING_MODEL=nomic-embed-text
```

**GPU Acceleration on macOS**: Ollama automatically uses Metal GPU acceleration on Apple Silicon (M1/M2/M3/M4) chips. The models will leverage the Neural Engine and GPU cores for optimal performance without any additional configuration. You can verify GPU usage by monitoring activity in Activity Monitor (look for high GPU utilization when running queries).

### ChromaDB Persistence

- Vector data persists in `./chroma_db/` (configurable)
- Use `pipeline.reset()` to clear all data
- Each collection is independent (useful for multiple projects)

### PDF Layout Processing

The `pdf_processor.py` module provides layout-aware PDF extraction:

**Key Features:**

- Extracts bounding boxes (x0, y0, x1, y1) for every text element
- Preserves font information (size, bold, italic styling)
- Determines position context (top/middle/bottom, left/center/right)
- Automatically detects likely headings based on font size and styling
- Groups words into logical text lines

**How It Helps:**
The vector database receives text annotated with layout metadata like:

```
[position:top-left | type:heading | size:16.0 | style:bold | bbox:[72,100,300,120]]
Document Title

[position:middle-left | size:12.0 | bbox:[72,150,500,165]]
This is the main body text...
```

This enables the LLM to understand that text in certain positions (headers, footers, sidebars) may have different semantic importance than body text, improving retrieval quality for layout-heavy documents like forms, invoices, or multi-column articles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronmak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aaronmak)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
