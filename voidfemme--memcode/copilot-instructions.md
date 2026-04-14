## memcode

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Run MemCode with Chainlit
chainlit run app/main.py

# Run in development mode with auto-reload
chainlit run app/main.py --watch
```

### Database Operations
```bash
# Generate database migration
python -m alembic revision --autogenerate -m "Description"

# Apply migrations
python -m alembic upgrade head

# Downgrade migration
python -m alembic downgrade -1
```

### Testing
```bash
# Run all tests
python -m pytest

# Run specific test file
python -m pytest tests/test_function_manager.py

# Run tests with coverage
python -m pytest --cov=services --cov=core --cov=data

# Run integration tests
python -m pytest tests/test_integration.py -v
```

### Code Quality
```bash
# Format code
python -m black .

# Sort imports
python -m isort .

# Type checking
python -m mypy services/ core/ data/

# Run all quality checks
python -m black . && python -m isort . && python -m mypy services/ core/ data/
```

## Architecture Overview

**MemCode** is an AI-powered coding assistant with intelligent memory and function management capabilities. The system uses a layered architecture:

### Core Components

**Application Layer (`app/`)**
- `main.py`: Chainlit application entry point with async event handlers
- Handles user interactions and coordinates between services

**Services Layer (`services/`)**
- `llm_service.py`: Claude integration with function calling tools (save_function, search_functions)
- `function_manager.py`: Function storage, retrieval, and versioning management
- `memory_manager.py`: Conversation memory with context retrieval
- `retrieval_service.py`: Enhanced search with semantic similarity and categorization

**Data Layer (`data/`)**
- `models.py`: SQLAlchemy models with versioning, testing, and performance tracking
- `repositories.py`: Data access patterns (planned)

**Core Infrastructure (`core/`)**
- `database.py`: Async database connection management
- `embeddings.py`: Vector embedding utilities

**Tools (`tools/`)**
- `execution.py`: Secure function execution with sandboxing, timeout, and resource limits
- `base.py`, `registry.py`: Extensible tool framework

### Key Features

**Function Versioning System**
- Each function has version tracking with base_function_id relationships
- Change summaries and author tracking
- Rollback capabilities and version comparison

**Secure Execution Environment**
- AST-based security analysis before execution
- Resource limits (memory, CPU time)
- Multiprocessing isolation with timeout protection
- Whitelist-based module/builtin restrictions

**Semantic Search & Retrieval**
- Sentence transformer embeddings for semantic similarity
- Hybrid search combining semantic and keyword matching
- Automatic function categorization and recommendations
- TF-IDF fallback for lightweight operations

**Testing Framework**
- Test case storage and execution tracking
- Performance metrics and success rate monitoring
- Regression testing for function updates

### Database Models

**Function Model** - Core entity with:
- Versioning fields (version, base_function_id, is_latest_version)
- Testing fields (test_cases, test_results, test metrics)
- Performance tracking (execution_count, avg_execution_time_ms)
- Security scoring (security_score, complexity_score)

**Supporting Models**:
- `FunctionExecution`: Execution history and performance metrics
- `FunctionDependency`: Inter-function dependency tracking
- `ConversationMemory`: Context-aware conversation storage

### Configuration

**Environment Variables**
- `ANTHROPIC_API_KEY`: Required for AI functionality
- `DATABASE_URL`: Database connection (SQLite dev, PostgreSQL prod)
- `DEBUG`: Debug mode toggle

**Dependencies**
- Chainlit for conversational UI
- SQLAlchemy with async support for database operations
- Sentence Transformers for semantic search
- Anthropic client for Claude integration

### Development Patterns

**Async-First Design**
- All database operations use async/await
- Service layer designed for concurrent operations
- Chainlit handlers are async

**Security-First Approach**
- Code execution happens in isolated processes
- Comprehensive security analysis before execution
- Resource usage monitoring and limits

**Modular Architecture**
- Clear separation of concerns between layers
- Dependency injection for testability
- Extensible tool registry system

### Testing Strategy

**Test Organization**
- Unit tests for individual services (`test_function_manager.py`)
- Integration tests for end-to-end workflows (`test_integration.py`)
- Service-specific test suites (`test_memory_manager.py`, `test_retrieval_service.py`)

**Key Test Areas**
- Function execution safety and sandboxing
- Semantic search accuracy and performance
- Version management and rollback functionality
- Database operations and migration integrity

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/voidfemme)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/voidfemme)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
