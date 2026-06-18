## bugviper

> This document provides essential information for AI agents working in the BugViper repository.

# AGENTS.md - BugViper Codebase Guide

This document provides essential information for AI agents working in the BugViper repository.

## Project Overview

BugViper is an AI-powered code review and repository intelligence platform. It ingests repositories into a Neo4j knowledge graph via Tree-sitter AST parsing and provides:
- Full-text and semantic code search
- LangGraph-powered PR review agent
- AI chat interface for codebase queries

**Tech Stack:**
- Backend: Python 3.13+, FastAPI, Neo4j, LangGraph
- Frontend: Next.js 16, React 19, TypeScript, TailwindCSS
- Package Manager: `uv` (Python), `npm` (frontend)

---

## Build, Lint, and Test Commands

### Python Backend

**Install dependencies:**
```bash
uv sync
```

**Format code:**
```bash
black .
```

**Lint:**
```bash
ruff check .
```

**Type check:**
```bash
mypy .
```

**Run tests:**
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov

# Run a specific test file
pytest tests/test_module.py

# Run a specific test function
pytest tests/test_module.py::test_function_name

# Run tests with verbose output
pytest -v
```

**Run development server:**
```bash
# API server only
uvicorn api.app:app --host 0.0.0.0 --port 8000 --reload

# Run all services (API, Ingestion, Lint, Frontend, Ngrok)
./start.sh
```

**Database setup:**
```bash
# Initialize Neo4j schema and indexes
curl -X POST http://localhost:8000/api/v1/ingest/setup
```

### Frontend (TypeScript/Next.js)

**Install dependencies:**
```bash
cd apps/frontend && npm install
```

**Lint:**
```bash
cd apps/frontend && npm run lint
```

**Build:**
```bash
cd apps/frontend && npm run build
```

**Development server:**
```bash
cd apps/frontend && npm run dev
# Runs on http://localhost:3000
```

---

## Code Style Guidelines

### Python

**Formatting:**
- Line length: 100 characters (configured in pyproject.toml)
- Use `black` for automatic formatting
- Use `ruff` for linting

**Imports:**
- Order: standard library → third-party → local imports
- Use absolute imports for project modules
- Example:
  ```python
  import logging
  import os
  from typing import Any, Dict, List, Optional
  
  from fastapi import APIRouter, Depends, HTTPException
  from neo4j import GraphDatabase
  
  from api.dependencies import get_neo4j_client
  from db.client import Neo4jClient
  ```

**Type hints:**
- Required for all function parameters and return values
- Use `Optional[T]` for optional parameters
- Use `list[T]`, `dict[str, Any]` (lowercase) for Python 3.13+
- Example:
  ```python
  def run_query(
      self, 
      query: str, 
      parameters: Optional[Dict[str, Any]] = None, 
      max_retries: int = 3
  ) -> Tuple[List[Any], Any, List[str]]:
  ```

**Naming conventions:**
- Functions/variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Private methods: `_leading_underscore`

**Docstrings:**
- Use triple-quoted docstrings with Args/Returns sections
- Example:
  ```python
  def search_code(query: str, limit: int = 30) -> Dict[str, Any]:
      """
      Search code in the Neo4j graph.
      
      Args:
          query: Search term or keyword
          limit: Maximum results to return
      
      Returns:
          Dictionary with 'results', 'total', and 'query' keys
      """
  ```

**Error handling:**
- Use specific exception types
- In FastAPI routes: raise `HTTPException(status_code=..., detail=...)`
- Log errors before re-raising
- Example:
  ```python
  try:
      results = query_service.search_code(query, repo_id=repo_id)
  except Exception as e:
      logger.error("Search failed: %s", e)
      raise HTTPException(status_code=500, detail=f"Search failed: {str(e)}")
  ```

**Logging:**
- Use `logging.getLogger(__name__)` pattern
- Example:
  ```python
  logger = logging.getLogger(__name__)
  logger.info("Connected to Neo4j database")
  logger.warning("Connection failed: %s", e)
  logger.error("Search failed: %s", str(e))
  ```

**API routes:**
- Use FastAPI's dependency injection for database connections
- Example:
  ```python
  router = APIRouter()
  
  def get_query_service(db: Neo4jClient = Depends(get_neo4j_client)) -> CodeSearchService:
      return CodeSearchService(db)
  
  @router.get("/search")
  async def search_code(
      query: str = Query(..., description="Search term"),
      limit: int = Query(30, description="Maximum results"),
      query_service: CodeSearchService = Depends(get_query_service),
  ) -> Dict[str, Any]:
  ```

### TypeScript/React

**Formatting:**
- Use ESLint with eslint-config-next
- No explicit `any` types (warned)
- Prefer functional components with arrow functions

**Imports:**
- Use path alias `@/` for imports from frontend root
- Use `import type` for type-only imports
- Example:
  ```typescript
  import type { Metadata } from "next";
  import { Toaster } from "@/components/ui/sonner";
  import { AuthProvider } from "@/lib/auth-context";
  ```

**Type definitions:**
- Define interfaces for all API responses
- Use strict null checking (strict mode enabled)
- Example:
  ```typescript
  export interface RepositoryStatistics {
    files: number;
    classes: number;
    functions: number;
    methods: number;
    lines: number;
    imports: number;
    languages: string[];
  }
  ```

**Error handling:**
- Use try/catch with async/await for API calls
- Handle null/undefined explicitly
- Example:
  ```typescript
  const token = await getFirebaseToken();
  if (token) {
    headers["Authorization"] = `Bearer ${token}`;
  }
  ```

**Components:**
- Use functional components with props typing
- Example:
  ```typescript
  export default function RootLayout({ children }: { children: React.ReactNode }) {
    return (
      <html lang="en" className="dark">
        {children}
      </html>
    );
  }
  ```

---

## Project Structure

```
api/                         # FastAPI backend
├── app.py                   # Entry point, routers, middleware
├── routers/
│   ├── ingestion.py         # Repository ingestion endpoints
│   ├── query.py             # Code search endpoints
│   ├── repository.py        # Repository management
│   └── webhook.py           # GitHub webhooks
└── services/
    ├── firebase_service.py  # Firebase integration
    ├── push_service.py      # Incremental updates
    └── review_service.py    # PR review pipeline (SIMPLIFIED)

code_review_agent/           # 3-Node LangGraph review agent (SIMPLIFIED)
├── agent_executor.py        # Run 3-node agent (Explorer → Reviewer → Summarizer)
├── context_builder.py       # Build markdown context for agent
├── utils.py                 # LLM loader (OpenRouter)
├── config.py                # Agent configuration
├── app.py                   # FastAPI app
├── nagent/                  # Core agent implementation
│   ├── ngraph.py            # 3-node graph definition
│   ├── ntools.py            # 19 code exploration tools
│   ├── nprompt.py           # 3 system prompts
│   ├── nstate.py            # State models (Pydantic)
│   ├── nrunner.py           # CLI runner
│   └── example_3node.py     # Example usage
└── models/
    └── agent_schemas.py     # Pydantic models for API

db/                          # Neo4j layer
├── client.py                # Connection management
├── queries.py               # CodeQueryService
└── schema.py                # Constraints + indexes

ingestion_service/           # Tree-sitter parsing
├── app.py                   # FastAPI service
└── routers/
    └── ingest.py            # Ingestion endpoints

common/                      # Shared utilities
├── embedder.py              # OpenRouter embeddings
├── diff_parser.py           # Unified diff parsing
├── debug_writer.py          # Write debug files
├── diff_line_mapper.py      # Map diff lines to file lines
├── github_client.py         # GitHub API client
├── firebase_models.py       # Firebase models
└── firebase_init.py         # Firebase initialization

apps/frontend/               # Next.js 16 app
├── app/(protected)/
│   ├── query/               # Search & analysis
│   └── repositories/        # Repo management
└── lib/
    └── api.ts               # API client
```

---

## Code Review Agent Architecture

BugViper uses a **3-node LangGraph agent** for code review:

### Architecture Flow

```
GitHub PR Webhook (@bugviper review)
    ↓
api/services/review_service.py
    ├─ Fetch PR data (diff, files, ASTs)
    ├─ Loop through each file:
    │   ├─ context_builder.py - Build markdown prompt
    │   └─ agent_executor.py - Run 3-node agent
    │       └─ ngraph.py - Execute:
    │           ├─ Explorer Node (ReAct loop with tools)
    │           ├─ Reviewer Node (Issues + Positives)
    │           └─ Summarizer Node (Walkthrough)
    ├─ Aggregate results
    ├─ Post GitHub comments
    └─ Save to Firestore
```

### Key Components

**1. review_service.py** (Entry Point)
- Orchestrates the entire review pipeline
- Fetches PR data (diff, files, ASTs)
- Calls agent for each file
- Aggregates results
- Posts GitHub comments

**2. context_builder.py** (Context Building)
- Builds markdown context from diff, code, AST
- Formats hunk ranges
- Renders code samples

**3. agent_executor.py** (Agent Runner)
- Builds the LangGraph
- Invokes 3-node agent
- Returns structured results
- Writes debug output

**4. ngraph.py** (Graph Definition)
- Node 1: Explorer - Investigates with tools (ReAct loop)
- Node 2: Reviewer - Generates issues and positives (structured output)
- Node 3: Summarizer - Generates walkthrough (structured output)

### Node Details

**Explorer Node:**
- Uses 19 code exploration tools (ntools.py)
- Investigates dependencies, complexity, security
- Bounded by MAX_TOOL_ROUNDS (default: 8)
- Accumulates evidence in messages

**Reviewer Node:**
- Reads full message history
- Calls LLM with `with_structured_output()`
- Output: `file_based_issues`, `file_based_positive_findings`
- Precise line numbers, confidence scoring

**Summarizer Node:**
- Generates narrative walkthrough
- Calls LLM with `with_structured_output()`
- Output: `file_based_walkthrough`

### Configuration

```python
# In .env
USE_3NODE_AGENT=true              # Enable 3-node agent (default: true)
MAX_TOOL_ROUNDS=8                 # Max tool calls per file
REVIEW_MODEL=anthropic/claude-sonnet-4-5  # LLM model
OPENROUTER_API_KEY=your-key       # API key
```

### Output Files

```
output/review-{timestamp}/
├── 01_diff.md                         # Raw diff
├── 02_parsed_files.json               # Parsed ASTs
├── 04_review_prompt_{filename}.md     # Agent input
├── 05_agent_output_{filename}.md       # Agent output (NEW)
├── 00_diff_parsing_debug.json         # Debug info
└── 05_aggregated.md                   # Final results
```

### Key Files

- `api/services/review_service.py` - Main pipeline (simplified)
- `code_review_agent/agent_executor.py` - Agent runner
- `code_review_agent/context_builder.py` - Context utilities
- `code_review_agent/nagent/ngraph.py` - Graph definition
- `code_review_agent/nagent/ntools.py` - 19 code exploration tools
- `code_review_agent/nagent/nprompt.py` - 3 system prompts
- `code_review_agent/nagent/nstate.py` - State models

---

## Key Patterns

**Python:**
- Dependency injection for services (FastAPI Depends)
- Pydantic models for request/response validation
- Context managers for database sessions
- Retry logic with exponential backoff for transient errors

**TypeScript:**
- Server-side auth in protected routes (AuthProvider wrapper)
- Centralized API client in `lib/api.ts`
- Firebase auth token injection on each request
- shadcn/ui components with Tailwind styling

---

## Testing Strategy

Currently no dedicated test suite is present in the repository. When adding tests:

**Python:**
- Use `pytest` as the test framework
- Place test files in `tests/` directory
- Name files as `test_<module>.py`
- Use pytest fixtures for common setup
- Run: `pytest tests/test_module.py::test_function_name -v`

**Testing the Code Review Agent:**
```bash
# Set environment
export OPENROUTER_API_KEY="your-key"
export NEO4J_URI="bolt://localhost:7687"

# Test imports
python -c "from api.services.review_service import review_pipeline; from code_review_agent.agent_executor import execute_review_agent; print('✅ Works!')"

# Run review (via webhook or programmatically)
# Comment "@bugviper review" on a PR
# Or run: uvicorn api.app:app --reload

# Check output
ls output/review-*/
# Files: 01_diff.md, 02_parsed_files.json, 04_review_prompt_*.md, 05_agent_output_*.md
```

**Frontend:**
- Consider Jest + React Testing Library
- Test file naming: `ComponentName.test.tsx`

---

## Environment Setup

Required environment variables (see `.env.example`):
- `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD` - Neo4j connection
- `OPENROUTER_API_KEY` - LLM and embeddings
- `GITHUB_APP_ID`, `GITHUB_PRIVATE_KEY_PATH`, `GITHUB_WEBHOOK_SECRET` - GitHub App
- `SERVICE_FILE_LOC` - Firebase service account
- `API_ALLOWED_ORIGINS` - CORS origins
- Optional: `ENABLE_LOGFIRE`, `LOGFIRE_TOKEN` for observability

**Agent Configuration:**
- `USE_3NODE_AGENT` - Enable 3-node agent (default: true)
- `REVIEW_MODEL` - LLM model for review (default: openai/gpt-4o-mini)
- `MAX_TOOL_ROUNDS` - Max tool calls per file (default: 8)

---

## Important Notes

1. **No comments in code** unless requested - code should be self-documenting
2. **Concise responses** - minimize output while maintaining quality
3. **Follow existing patterns** - check neighboring files for conventions
4. **Security first** - never expose secrets or API keys
5. **Type safety** - strict typing in both Python and TypeScript
6. **Line length** - 100 characters for Python (enforced by black)
7. **Database** - Always use dependency injection for Neo4j client

**Agent Architecture:**
- The 3-node agent (Explorer → Reviewer → Summarizer) is the default architecture
- Each node has a single responsibility for better separation of concerns
- Tools are bounded by MAX_TOOL_ROUNDS (default: 8) for cost control
- Structured output via Pydantic models ensures type safety

**Cost & Performance:**
- Typical review: ~$0.12 per file with claude-sonnet-4-5
- Tool rounds: 5-8 per file
- Tokens: ~6,500 per file review
- Reduce MAX_TOOL_ROUNDS for simple files to save cost

---
> Source: [MabudAlam/BugViper](https://github.com/MabudAlam/BugViper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
