## kite-rag

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KITE-RAG (Knowledge Graph Integrated Text-enhanced Retrieval-Augmented Generation) is an advanced RAG system that combines knowledge graphs with textual information for context-aware question answering. The system has two main components:

1. **Knowledge Base Construction**: Processes documents to extract entities, relationships, and build knowledge graphs
2. **Hybrid Graph-Text QA**: Performs query decomposition and multi-modal retrieval for answer generation

## Development Commands

### Environment Setup
```bash
# Install dependencies using pip
pip install -r requirements.txt

# Or install as editable package with development dependencies
pip install -e ".[dev]"

# Install with GPU support
pip install -e ".[gpu]"
```

### Running the Application

```bash
# Start Streamlit interface  
streamlit run st_main.py
```

### Testing
```bash
# Run all tests
pytest

# Run specific test modules
pytest tests/index_test.py
pytest tests/search_test.py

# Run with coverage
pytest --cov=src/kite_rag tests/
```

### Code Quality
```bash
# Lint code
pylint src/kite_rag/

# Format code
black src/kite_rag/ tests/
isort src/kite_rag/ tests/

# Type checking
mypy src/kite_rag/
```

## Core Architecture

### Package Structure
- `src/kite_rag/`: Main package
  - `types/`: Core data models (Entity, Relation, Chunk, Image)
  - `index/`: Knowledge base construction pipeline
    - `pipe.py`: Main text processing pipeline
    - `text.py`: Entity-relation extraction
    - `mmodal.py`: Multi-modal processing
    - `deduplicate.py`: Entity deduplication
  - `retrieval/`: Query processing and search
    - `search.py`: Multi-modal search implementation
    - `agents.py`: Query decomposition agents
    - `merge.py`: Result merging logic
  - `storage/`: Memory storage management
  - `gui/`: Streamlit web interface
  - `utils/`: Shared utilities (logging, LLM, helpers)

### Key Design Patterns
- **Pydantic Models**: All data types use Pydantic for validation (Entity, Relation, Chunk, Image)
- **Pipeline Architecture**: Text processing follows a pipeline pattern in `index/pipe.py`
- **Streamlit UI**: User-friendly Streamlit interface (`st_main.py`)
- **Async Processing**: Some operations use asyncio for concurrent processing
- **Memory Storage**: In-memory storage with pickle serialization for entities/relations

### Dependencies
- **Neo4j**: Knowledge graph database (requires running Neo4j instance)
- **ChromaDB**: Vector database for text embeddings
- **OpenAI API**: LLM integration for entity extraction and QA
- **sentence-transformers**: Text embeddings (uses all-MiniLM-L6-v2 model)
- **marker-pdf**: PDF processing and conversion to markdown

## Environment Configuration

The system requires a `.env` file with:
```env
BASE_URL=https://api.openai.com/v1
API_KEY=your-api-key
LLM_MODEL=gpt-4o-mini
NEO4J_URI=neo4j://127.0.0.1:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=yourStrongPassword
```

## Testing Strategy

- Test files follow `*_test.py` naming convention
- Tests are located in `tests/` directory
- Key test modules:
  - `index_test.py`: Knowledge base construction
  - `search_test.py`: Retrieval functionality
  - `mmodal_test.py`: Multi-modal processing
  - `parser_test.py`: Text parsing

## Code Style

- Uses Black formatter with 88-character line length
- PyLint configuration in `.pylintrc` with relaxed docstring requirements
- Type hints required (mypy configuration in pyproject.toml)
- Import sorting with isort using Black profile

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luhsinyuan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
