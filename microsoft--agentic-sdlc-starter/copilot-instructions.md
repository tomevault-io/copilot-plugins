## agentic-sdlc-starter

> {{PROJECT_NAME}} — contract intelligence system. Analysts query a Neo4j graph database of payer contracts using natural language. System classifies queries, executes Cypher or vector search, and streams responses.

# Copilot Instructions — Contract Intelligence Pipeline

## Project Overview
{{PROJECT_NAME}} — contract intelligence system. Analysts query a Neo4j graph database of payer contracts using natural language. System classifies queries, executes Cypher or vector search, and streams responses.

## Architecture

### Tech Stack
- **Backend:** Python 3.12+, FastAPI, uvicorn
- **Frontend:** React 18+, TypeScript, Tailwind CSS
- **Database:** Neo4j (graph DB, Cypher queries)
- **LLM:** Google Gemini 2.5 Flash (query classification, summarization)
- **Communication:** WebSocket (real-time streaming)
- **Testing:** pytest (backend), vitest (frontend)

### Clean Architecture Layers
```
Domain (entities, value objects)        — no external dependencies
Application (use cases, interfaces)     — depends on Domain only
Infrastructure (Neo4j, Gemini, WS)      — implements Application interfaces
API (FastAPI routes, WebSocket)          — thin, delegates to Application
```

**Dependency rule:** Inner layers NEVER import from outer layers. Use dependency injection.

### Project Structure
```
src/
├── api/
│   ├── main.py                    # FastAPI app, CORS, lifespan
│   ├── routes/
│   │   ├── chat.py                # WebSocket /ws/chat endpoint
│   │   └── health.py              # /health endpoint
│   └── dependencies.py            # DI container
├── application/
│   ├── interfaces/
│   │   ├── llm_client.py          # Abstract LLM interface
│   │   ├── graph_client.py        # Abstract graph DB interface
│   │   └── query_classifier.py    # Abstract classifier interface
│   └── use_cases/
│       ├── process_query.py       # Main orchestrator
│       ├── classify_query.py      # Query classification
│       ├── execute_cypher.py      # Cypher query execution
│       └── format_response.py     # Response formatting
├── domain/
│   ├── models/
│   │   ├── query.py               # Query, Classification entities
│   │   ├── contract.py            # Contract, Amendment, Correspondence
│   │   └── response.py            # Response, Citation entities
│   └── enums/
│       └── query_type.py          # 6 query types enum
├── infrastructure/
│   ├── neo4j_client.py            # Neo4j driver wrapper
│   ├── gemini_client.py           # Gemini API wrapper
│   └── cypher_templates.py        # Cypher query templates
└── tests/
    ├── unit/
    ├── integration/
    └── conftest.py
```

## Coding Standards

### Python
- Type hints on ALL function signatures (parameters and return types)
- Pydantic models for all data structures (BaseModel, not dataclasses)
- Async/await for all I/O operations (DB queries, LLM calls, WebSocket)
- Structured logging with `structlog` (not print statements)
- Error handling: custom exception hierarchy, never catch bare `Exception`
- Docstrings: Google style, required on all public methods

### TypeScript/React
- Functional components only (no class components)
- TypeScript strict mode (no `any` types)
- Custom hooks for WebSocket connection and state management
- Tailwind CSS for styling (no CSS modules)
- Error boundaries for resilience

### General
- No hardcoded values — use environment variables via pydantic-settings
- All external calls must have timeout and retry logic
- Every public function must have at least one unit test

## Industry-Specific Guardrails

### {{SENSITIVE_DATA_LABEL}} Rules (MANDATORY)
- **PII audit logging:** ALWAYS enabled (`DATA_CLASSIFICATION = "Confidential"`)
- **No PHI in logs:** Never log contract content, sensitive data, or query results at INFO level
- **Structured audit trail:** Every query must be logged with session_id, timestamp, query_type, but NOT the actual content
- **Data classification:** All contract data is Confidential. Treat accordingly.

### Security
- Input validation on all user queries (max 2000 chars, sanitize)
- No raw string interpolation in Cypher queries (use parameterized queries ONLY)
- WebSocket authentication required (session token)
- CORS restricted to known origins
- Rate limiting on WebSocket connections

## Neo4j Data Model

### Nodes
- `Contract` — contract_id, contract_name, contract_status, document_eff_start_date, document_eff_end_date, evergreen, negotiation_id
- `Amendment` — document_id, title, document_eff_start_date, description
- `Correspondence` — document_id, title, document_eff_start_date, description
- `MajorPayor` — major_payor (e.g., "590 - Humana")
- `Facility` — facility_name

### Relationships
- `(Contract)-[:HAS_AMENDMENT]->(Amendment)`
- `(Contract)-[:HAS_CORRESPONDENCE]->(Correspondence)`
- `(Contract)-[:HAS_PAYOR]->(MajorPayor)`
- `(Contract)-[:HAS_FACILITY]->(Facility)`

## Query Classification (6 Types)

| Type | Strategy | Description |
|------|----------|-------------|
| `find_clauses_by_keyword` | Dual-Path RAG | Search clause content |
| `list_all_amendments` | Cypher Only | List amendment documents |
| `list_all_correspondence` | Cypher Only | List correspondence documents |
| `get_contract_property` | Cypher Only | Get specific metadata field |
| `summarize_contract` | LLM Summary | Contract overview |
| `other` | Vector Search | General questions needing context |

## Zero-Assumption Principle
**NEVER assume when ambiguity exists.** If a requirement is unclear:
1. Flag the ambiguity explicitly
2. Present 2-3 options with tradeoffs
3. Wait for engineer confirmation before proceeding
4. Document the decision in code comments

This applies to architecture decisions, data model choices, and implementation details.

## Testing Requirements
- Unit test coverage: 80%+ per module
- Integration tests: Full round-trip for each query type
- Test data: Use fixtures, never depend on live Neo4j
- Mock external services (Gemini, Neo4j) in unit tests

---
> Source: [microsoft/agentic-sdlc-starter](https://github.com/microsoft/agentic-sdlc-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
