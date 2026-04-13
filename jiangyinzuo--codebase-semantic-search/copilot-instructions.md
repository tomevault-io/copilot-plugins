## codebase-semantic-search

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Building and Installation:**
```bash
# Build the package
python -m build

# Install locally
pip install .

# Install in development mode (editable)
pip install -e .

# Install with test dependencies
pip install -e .[test]
```

**Database Setup:**
```bash
# Start PostgreSQL with pgvector
docker run --name codebase-indexing -e POSTGRES_HOST_AUTH_METHOD=trust -p 5439:5432 -d pgvector/pgvector:0.8.0-pg17

# Create database and tables
createdb -h 127.0.0.1 -p 5439 -U postgres codebase_indexing
psql -h 127.0.0.1 -p 5439 -U postgres -d codebase_indexing -f create_tables.sql -v dim=1024
```

**Testing:**
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_indexing.py

# Run specific test function
pytest tests/test_indexing.py::test_git_changes_added_files -v

# Run tests with coverage
pytest --cov=src/codebase

# Run integration tests (requires PostgreSQL with pgvector)
python run_integration_tests.py --with-db

# Run unit tests only (no database required)
python run_integration_tests.py --unit-only

# Run MCP server tests
pytest tests/test_mcp_server.py -v --asyncio-mode=auto
```

**CLI Usage:**
```bash
# Index files
codebase index -a "file1.py file2.cpp"

# Search codebase
codebase search -q "function that handles authentication"

# Show configuration
codebase config

# Start MCP server for AI assistants
codebase-mcp
```

## Architecture Overview

**Core Components:**
- `src/codebase/cli.py`: Command-line interface with subcommands for index/search/config
- `src/codebase/config.py`: Hierarchical configuration system (global + project-specific JSONC)
- `src/codebase/indexing.py`: File processing and Tree-sitter based code chunking
- `src/codebase/mcp_server.py`: MCP server for AI assistant integration
- `src/codebase/model_provider.py`: Embedding model abstraction (OpenAI API + local sentence-transformers)
- `src/codebase/pgvector.py`: PostgreSQL with pgvector database operations
- `src/codebase/search.py`: Semantic search functionality
- `src/codebase/ts_chunk.py`: Tree-sitter language processing utilities

**Key Dependencies:**
- PostgreSQL with pgvector extension
- sentence-transformers for local embeddings
- tree-sitter for language parsing
- jsonc-parser for configuration files
- psycopg2 for PostgreSQL connectivity
- numpy for vector operations
- tabulate for CLI output formatting
- mcp[cli] for MCP server functionality

**Neovim Integration:**
- `lua/codebase-semantic-search.lua`: Main plugin implementation
- `plugin/codebase-semantic-search.lua`: Plugin entry point
- Uses plenary.nvim for async job management
- Provides interactive search panel with keymaps

## Configuration System

Supports hierarchical configuration merging:
1. Default configuration in `config.py`
2. Global config: `~/.config/codebase/config.jsonc`
3. Project config: `.codebase/config.jsonc` (recursively searched upward)

**Key Configuration Options:**
- `pgvector`: Database connection settings
- `model_provider`: "openai" or "sentence_transformer"
- `model`: Path to local model or OpenAI-compatible endpoint
- `openai.url`: API endpoint for OpenAI-compatible services

## Development Notes

- **Testing**: Basic pytest test suite exists in `tests/` directory with mocking support, including MCP server integration tests
- **Python Version**: Requires Python 3.10+ (uses match statements, type hints)
- **Packaging**: Built with setuptools via pyproject.toml
- **Neovim Integration**: Requires plenary.nvim dependency
- **Tree-sitter**: Grammars need to be installed separately for language support
- **Configuration**: Uses JSONC format with hierarchical merging (global + project)
- **Development Workflow**: After modifying source code, always run `pip install -e .` to apply changes
- **Integration Testing**: Use `python run_integration_tests.py --with-db` for database-dependent tests

**File Structure:**
```
codebase-semantic-search/
├── src/codebase/          # Python package source
├── lua/                   # Neovim plugin implementation
├── plugin/               # Neovim plugin entry point
├── create_tables.sql     # Database schema
└── pyproject.toml       # Python package configuration
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jiangyinzuo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jiangyinzuo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
