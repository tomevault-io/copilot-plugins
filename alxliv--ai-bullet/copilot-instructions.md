## ai-bullet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Bullet is a Retrieval-Augmented Generation (RAG) chatbot that provides intelligent documentation search and code understanding for software projects. It combines OpenAI's embedding models with ChromaDB for semantic search across PDFs, documentation files, and C++ source code. The Bullet3 physics library serves as the testbed example.

## Architecture

### Core Components

**app.py** - Main FastAPI application
- RAG-powered chat interface with retrieval from ChromaDB collections
- Modern async lifespan management for OpenAI client initialization
- Session management with per-user chat histories
- Streaming chat endpoint (`/chat`) using Server-Sent Events (SSE)
- Automatic chat history saving to `saved_chats/` directory
- Static file serving for docs (`/docs`), source code (`/src`), and examples (`/examples`)
- LaTeX rendering support with KaTeX (four delimiter types)
- Health check and model listing endpoints

**retriever.py** - RAG retrieval engine
- Multi-collection ChromaDB querying with reciprocal rank fusion
- Optional Maximal Marginal Relevance (MMR) diversification
- Optional LLM-based re-ranking using GPT models
- Token-budget aware context building using tiktoken
- Configurable system/user prompt templates via `RetrieverConfig`

**updatedb_docs.py** - Documentation ingestion script
- Processes PDFs, DOCX, Markdown, and text files
- Token-aware chunking with configurable overlap
- Batch embedding with OpenAI API
- Stores in ChromaDB collection (typically "bullet_docs")

**updatedb_code.py** - C++ code ingestion script
- Uses Lizard for C++ parsing (functions, classes, structs)
- Extracts leading comments for context
- Chunks oversized functions by token budget
- Stores in ChromaDB collection (typically "cpp_code")

**config.py** - Central configuration
- Paths to documentation, source code, and examples
- ChromaDB directory and embedding model selection
- Chunk sizes and overlap settings
- Use `config_win.py` (Windows) or `config_posix.py` (Linux) as templates

### Key Design Patterns

**RAG Pipeline Flow**:
1. User query → embed with OpenAI
2. Retrieve from multiple ChromaDB collections (code + docs)
3. Fuse results using reciprocal rank fusion or score-based fusion
4. Optional MMR diversification or LLM re-ranking
5. Build context within token budget
6. Generate response with OpenAI streaming

**Session Management**:
- Sessions stored in-memory dict: `{session_id: {"messages": [...], "model": "...", "username": "..."}}`
- Session IDs: `username_<8-char-random-id>`
- Automatic cleanup when > 100 sessions (keeps 50 most recent)
- Chat histories auto-saved to `saved_chats/<session_id>.json`

**OS-Agnostic Path System**:
File paths in ChromaDB use variable-based encoding for cross-platform compatibility:
- `{DOCS}/subfolder/file.pdf` - Documentation files
- `{SRC}/path/to/code.cpp` - Source code files
- `{EXAMPLES}/example/main.cpp` - Example files

The curly brace format `{VAR}` is used to avoid collisions with LaTeX math notation (`$...$`) commonly found in technical documentation. Legacy `$VAR$` format is still supported for backward compatibility. See `path_utils.py` for encoding/decoding logic and `PATH_SYSTEM.md` for detailed documentation.

## Development Commands

### Environment Setup
```bash
# Install uv (faster than pip)
pip install uv

# Create virtual environment
uv venv
.venv\Scripts\activate  # Windows
source .venv/bin/activate  # Linux/macOS

# Install dependencies
uv pip install -r requirements.txt
```

### Configuration
```bash
# Copy example environment file
cp .env_example .env

# Edit .env with your settings:
# - OPENAI_API_KEY: Your OpenAI API key
# - USERS_DB: username:password pairs (e.g., admin:12345,user01:demo)
# - ANONYMIZED_TELEMETRY: False
# - CHROMA_TELEMETRY: False

# Configure paths in config.py
# Set DOCUMENTS_PATH, SOURCES_PATH, EXAMPLES_PATH to your data directories
# Use config_win.py or config_posix.py as template
```

### Database Population
```bash
# Process documentation files (PDFs, DOCX, Markdown, text)
python updatedb_docs.py

# Process C++ source code
python updatedb_code.py

# Migrate existing database to OS-agnostic path encoding (if needed)
python migrate_paths.py --dry-run  # Preview changes
python migrate_paths.py            # Apply migration

# Test retrieval standalone
python retriever.py "How do I create a rigid body in Bullet3?"
```

### Running the Application
```bash
# Main application (recommended)
python app.py

# Alternative using uvicorn directly
uvicorn app:app --host 0.0.0.0 --port 8000 --reload

# For debugging with VS Code
# Use the "Debug FastAPI" configuration in .vscode/launch.json
```

## API Endpoints

- `GET /` - Main chat interface with RAG-powered responses
- `POST /chat` - Streaming chat endpoint (JSON: `{"message": "...", "model": "..."}`)
- `GET /health` - Health check with OpenAI configuration status
- `GET /models` - List available OpenAI models
- Static mounts: `/docs`, `/src`, `/examples` - Serve documentation and source files

## Important Development Notes

### Path Management System
The codebase uses an OS-agnostic path encoding system (`path_utils.py`):
- **Encoding**: Absolute paths → variable-based paths (e.g., `D:/Work/docs/file.pdf` → `$DOCS$/file.pdf`)
- **Decoding**: Variable-based paths → absolute paths for current OS
- **Variables**: `$DOCS$`, `$SRC$`, `$EXAMPLES$` map to paths in `config.py`
- **Storage**: All paths in ChromaDB metadata use encoded format
- **Migration**: Use `migrate_paths.py` to convert existing databases

This enables database portability across Windows, Linux, and macOS systems.

### Retriever Configuration
The `RetrieverConfig` class in `retriever.py` controls RAG behavior:
- `use_mmr`: Enable Maximal Marginal Relevance (default: True)
- `use_llm_rerank`: Use LLM for re-ranking instead of MMR (default: False)
- `max_context_tokens`: Token budget for context (default: 6000)
- `max_snippets`: Maximum number of snippets to include (default: 12)
- `system_template` vs `system_template_full`: Context-only vs full-knowledge prompts

### Model Selection
- GPT-5 models use `max_completion_tokens` parameter
- Other models use `max_tokens` parameter
- Temperature set to 1 for `-mini` and `-nano` models, 0 otherwise
- O1 models filter out system messages before sending to API

### LaTeX Rendering
The frontend supports four delimiter types:
- `$...$` and `$$...$$` (standard)
- `\(...\)` and `\[...\]` (LaTeX-style)

Math expressions are protected with placeholders before markdown parsing to avoid conflicts.

### Security Considerations
- CORS configured for localhost origins only (restrict to specific origins for production)
- API keys loaded from environment variables only
- Session cleanup prevents memory exhaustion (keeps 50 most recent when > 100 sessions)
- All user input validated through Pydantic models

### Testing and Debugging
- Use `retriever.py` standalone to test retrieval without web interface
- Check `saved_chats/` directory for conversation logs
- Enable uvicorn logging with `--log-level debug`
- Test LaTeX rendering with questions involving mathematical formulas

### Known Issues
1. **Session storage**: In-memory sessions lost on restart. Consider persistent storage for production.
2. **Source score filtering**: Minimum relevance thresholds (0.25 and 0.08) may need tuning for different datasets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alxliv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
