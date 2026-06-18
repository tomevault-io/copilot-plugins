## sql-template-agent

> A FastAPI backend for automotive trace analysis that enables engineers to analyze vehicle trace data through natural language queries. The system uses a ReAct agent that autonomously picks templates or generates SQL to answer queries.

# Trace Analysis API - Project Context

## Project Overview

A FastAPI backend for automotive trace analysis that enables engineers to analyze vehicle trace data through natural language queries. The system uses a ReAct agent that autonomously picks templates or generates SQL to answer queries.

**Core Workflow:** Ticket ID → Generate Data → Ask Agent Questions

---

## Architecture Principles

### 1. **Simple > Complex**
- Flat file structure, no unnecessary nesting
- Each file has ONE responsibility
- No premature abstractions

### 2. **Agent-First**
- Single entry point: the ReAct agent (`POST /analyze`)
- Agent autonomously searches templates or writes SQL
- Template search and execution kept as secondary endpoints

### 3. **Extensible by Design**
- Add new templates without touching core code
- Templates are self-contained modules

### 4. **Type-Safe**
- Pydantic models everywhere
- Request/response validation automatic
- Clear contracts between components

---

## High-Level Architecture

```
┌─────────────────────────────────────────────┐
│  FastAPI REST API                            │
│                                              │
│  Primary:                                    │
│  • /analyze          - Agent analysis       │
│  • /generate-data    - Sample data gen      │
│  • /schema           - DB schema info       │
│  • /tables/*         - Table browsing       │
│                                              │
│  Secondary (kept for later):                 │
│  • /discover         - Find templates       │
└──────────────┬──────────────────────────────┘
               │
┌──────────────┴──────────────────────────────┐
│  Agent (LangGraph ReAct)                     │
│  3 tools: search_templates,                  │
│           execute_template, execute_sql      │
└──────────────┬──────────────────────────────┘
               │
┌──────────────┴──────────────────────────────┐
│  Data & Integration Layer                    │
│  • DataManager       - PostgreSQL operations│
│  • TemplateSearch    - Weaviate hybrid search│
│  • LLMClient         - OpenAI/Anthropic/Ollama│
│  • JiraClient        - Ticket context       │
└──────────────────────────────────────────────┘
```

---

## Technology Stack

### Core Framework
- **FastAPI** (0.115+) - Modern async web framework
- **Pydantic** (2.9+) - Data validation and settings
- **Uvicorn** - ASGI server

### Data & SQL
- **PostgreSQL** - Analytical SQL database (per-ticket schema isolation)
- **psycopg** (3.x) - PostgreSQL driver with connection pooling
- **Pandas** (2.2+) - Data manipulation (cursor-based fetching, no `pd.read_sql`)

### LLM & AI
- **LangChain** - LLM abstraction (OpenAI, Anthropic, Ollama)
- **LangGraph** - ReAct agent loop

### Search
- **Weaviate** - Hybrid vector + keyword search for template discovery
- **Ollama** - Local embeddings (`nomic-embed-text`)

### Frontend
- **Next.js 16** + **React 19** + **TailwindCSS**
- **Axios** for API calls
- **react-markdown** for rendering agent responses (with prose CSS styling for code blocks, tables, blockquotes, lists)
- Sidebar + chat layout (not tabs) — ticket list in sidebar, chat in main area
- Inline tool results show SQL, data tables, template search cards, and source table name chips extracted from SQL

### Development
- **uv** - Fast Python package manager
- **Pytest** - Testing
- **Black** - Code formatting
- **Ruff** - Fast linting

---

## Project Structure

```
backend/
└── app/
    ├── main.py              # FastAPI app, lifespan, lazy singletons
    ├── config.py            # Settings (env-based)
    ├── routes.py            # All API routes
    ├── agent.py             # LangGraph ReAct agent, tools, prompt assembly
    ├── llm.py               # LLM client (OpenAI/Anthropic/Ollama)
    ├── database.py          # PostgreSQL operations (schema isolation)
    ├── search.py            # Weaviate hybrid search
    ├── jira.py              # Jira ticket context (in-memory)
    ├── models.py            # All request/response Pydantic models
    ├── data.py              # Sample data generators
    ├── skills/              # Agent skill files loaded into system prompt
    │   ├── template_analysis.md
    │   ├── freehand_sql.md
    │   └── iterative_analysis.md
    └── templates/           # SQL template system
        ├── base.py          # SQLTemplate model
        ├── library.py       # Template registry
        └── registry/        # Individual templates
            ├── error_lookup.py
            ├── time_window.py
            ├── count_occurrences.py
            ├── someip_errors.py
            ├── voltage_anomaly.py
            └── ecu_communication.py

react_frontend/
└── app/
    ├── page.tsx                    # Root page (sidebar + chat layout)
    ├── layout.tsx                  # Next.js layout
    ├── globals.css                 # Theme variables, prose styling, animations
    ├── lib/
    │   ├── api.ts                  # Backend API client
    │   ├── attachments.ts          # Attachment workflow helpers
    │   └── parseToolOutput.ts      # Tool result parser + SQL table extraction
    ├── hooks/
    │   └── useTicketDetail.ts      # Ticket detail data hook
    ├── types/index.ts              # TypeScript interfaces (ToolResult, ParsedTable, etc.)
    └── components/
        ├── AppShell.tsx            # Top-level sidebar + chat shell
        ├── DatabaseExplorer.tsx    # Table browser with search
        ├── chat/
        │   ├── ChatPanel.tsx       # Message list, streaming, conversation history
        │   ├── ChatInput.tsx       # Message input bar
        │   └── InlineToolResult.tsx # Tool result cards (tables, SQL, template search)
        └── sidebar/
            ├── Sidebar.tsx         # Sidebar container
            ├── SidebarHeader.tsx   # Logo / branding
            ├── SidebarNav.tsx      # Navigation links
            ├── TicketList.tsx      # Ticket browser
            ├── TicketContext.tsx    # Active ticket context display
            ├── TicketDetailModal.tsx # Ticket detail overlay
            └── AttachmentLoader.tsx # File attachment UI
```

---

## Key Design Patterns

### 1. **Agent-First Architecture**
The ReAct agent is the primary user-facing interface. It has 3 tools:
- `search_templates` - Find relevant SQL templates via Weaviate
- `execute_template` - Fill and run a template with parameters
- `execute_sql` - Run freehand SQL directly

Skills loaded from `.md` files guide the agent's reasoning.

### 2. **Lazy Singletons**
All heavy services (DB pool, Weaviate, LLM) are created lazily via `@lru_cache` in `main.py`, only initialized at startup — not at import time. This enables clean testing.

### 3. **Module-Level Dependencies**
Routes receive shared singletons via `init_dependencies()` called from the lifespan. No FastAPI `Depends()` for singletons — keeps routes simple.

### 4. **Template Pattern**
Templates define SQL structure, parameter schemas, domain knowledge, and example prompts. The agent uses them as tools.

---

## Coding Standards

### Python Style
- **Black** formatting (line length: 88)
- **Type hints** everywhere
- **Async/await** for I/O operations
- **Logging** via `logging.getLogger(__name__)` — no wrapper

### Error Handling
```python
try:
    result = await executor.execute(...)
    return result
except ValueError as e:
    raise HTTPException(status_code=400, detail=str(e))
except Exception as e:
    logger.error(f"Execution failed: {e}", exc_info=True)
    raise HTTPException(status_code=500, detail=str(e))
```

---

## Data Flow

### Primary: Agent Analysis
```
1. User query arrives at POST /analyze
2. Fetch Jira context + DB schema
3. Load skill files into system prompt
4. Create ReAct agent with 3 tools
5. Agent reasons, calls tools, observes results
6. Agent loops until satisfied
7. Return: answer + steps + timing
```

### Setup: Data Generation
```
1. POST /generate-data with ticket_id
2. Generate 4 DataFrames (trace_data, ecu_info, someip_messages, someip_services)
3. Generate correlated Jira ticket
4. Store in PostgreSQL schema scoped to ticket
5. Register Jira context for agent use
```

---

## Environment Configuration

### Required Settings (.env)
```bash
LLM_PROVIDER=openai          # or anthropic, ollama
LLM_MODEL=gpt-4
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_DOCKER_URL=http://host.docker.internal:11434
EMBEDDING_MODEL=nomic-embed-text
WEAVIATE_URL=http://localhost:8080
PG_HOST=localhost
PG_PORT=5432
PG_USER=postgres
PG_PASSWORD=...
PG_DATABASE=traces
LOG_LEVEL=INFO
```

---

## Development Workflow

### Setup
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
cd backend && uv sync
```

### Run
```bash
docker compose up -d                              # Weaviate + PostgreSQL
docker exec ollama ollama pull nomic-embed-text    # embeddings
cd backend && uv run uvicorn app.main:app --reload
cd react_frontend && npm run dev
```

### Test
```bash
cd backend && uv run pytest tests/ -v
cd react_frontend && npm run build
```

### Format
```bash
cd backend
uv run black app/ tests/
uv run ruff check app/ tests/
```

---

## Extension Points

### Adding a New Template
1. Create file in `app/templates/registry/`
2. Define `SQLTemplate` with parameters
3. Register in `TemplateLibrary._load_templates()`
4. Done — automatically available to agent

### Adding a New Endpoint
1. Add route function to `app/routes.py`
2. Define request/response models in `app/models.py`

---

## Key Principles Summary

1. **Simplicity** - Flat structure, no unnecessary abstractions
2. **Agent-first** - Agent is the primary interface
3. **Type Safety** - Pydantic models everywhere
4. **Testability** - Lazy singletons, easy to mock
5. **Extensibility** - Add templates without changing core

---
> Source: [BastianSpatz/sql_template_agent](https://github.com/BastianSpatz/sql_template_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
