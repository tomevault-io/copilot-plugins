## opencodeagent

> Multi-agent visualization platform for monitoring OpenCode plugin events:

# Multi-Agent Visualization Platform - Agent Guidelines

## Project Overview

Multi-agent visualization platform for monitoring OpenCode plugin events:
- **Backend**: Python 3.11+ (FastAPI, Pydantic, SQLite/PostgreSQL)
- **Frontend**: React 18+ (TypeScript, ReactFlow, Zustand, dayjs)
- **Storage**: SQLite (development), PostgreSQL/TimescaleDB (production), Redis (cache)
- **Plugin**: OpenCode visualization plugin in `plugins/opencode-viz-plugin/`

## Build/Lint/Test Commands

### Backend (Python)

```bash
# Install dependencies
pip install -r requirements.txt && pip install -r requirements-dev.txt

# Development server (local)
cd backend && uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Development server (Docker)
docker-compose up -d backend

# Tests
pytest                                    # All tests
pytest tests/test_trace_store.py          # Single file
pytest tests/test_trace_store.py::TestTraceStore::test_save_and_get_trace  # Single test
pytest --cov=app --cov-report=html        # With coverage
pytest -v                                 # Verbose output

# Code quality
ruff check app/                           # Lint
ruff check app/ --fix                     # Auto-fix lint issues
black app/                                # Format code
isort app/                                # Sort imports
mypy app/                                 # Type check

# Run all checks before commit
ruff check app/ && black app/ --check && pytest
```

### Frontend (TypeScript/React)

```bash
# Install and run
cd frontend && npm install
npm start                                 # Development server (port 3000)
npm run build                             # Production build

# Tests
npm test                                  # All tests (watch mode)
npm test -- --watchAll=false              # Single run
npm test -- FlowCanvas.test.tsx           # Single file
npm test -- --testNamePattern="renders"   # Pattern match

# Note: This project uses react-scripts (CRA)
# No separate lint/type-check scripts configured
```

### Docker (Full Stack)

```bash
docker-compose up -d --build              # Build and start all services
docker-compose up -d backend              # Start only backend
docker-compose logs -f backend            # Follow backend logs
docker-compose restart backend            # Restart backend after code changes
docker-compose down -v                    # Stop and clean volumes
```

### API Endpoints

```bash
curl http://localhost:8000/health         # Health check
curl http://localhost:8000/api/v1/traces  # List traces
curl http://localhost:8000/api/v1/events  # List events
```

## Code Style Guidelines

### Python (Backend)

**Imports** (in this order, separated by blank lines):
```python
# 1. Standard library
import asyncio
import json
import logging
from datetime import datetime
from typing import List, Optional, Dict, Any

# 2. Third-party
from fastapi import APIRouter, HTTPException
from pydantic import BaseModel, Field

# 3. Local imports
from app.models import AgentTrace, TraceStatus
from app.storage.trace_store import TraceStore
```

**Formatting**:
- Black (88 character line length)
- isort for import sorting
- ruff for linting
- Double quotes for strings

**Type Hints** (always required):
```python
async def save_trace(self, trace: AgentTrace) -> bool:
    ...

def get_agent(self, agent_id: str) -> Optional[Agent]:
    ...

def process_messages(self, messages: List[Dict[str, Any]]) -> None:
    ...
```

**Naming Conventions**:
- Classes: `PascalCase` (e.g., `AgentTrace`, `TraceStore`, `EventStore`)
- Functions/Variables: `snake_case` (e.g., `save_trace`, `agent_id`, `event_store`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`, `DEFAULT_TIMEOUT`)
- Private methods: `_leading_underscore` (e.g., `_init_db`, `_row_to_event`)
- Pydantic models: Suffix with purpose (e.g., `AgentExecuteRequest`, `AgentExecuteResponse`)

**Error Handling**:
```python
# Use HTTPException for API errors
from fastapi import HTTPException

raise HTTPException(status_code=404, detail="Agent not found")
raise HTTPException(status_code=500, detail=str(e))

# Log errors with context
logger.error("Trace save failed", extra={
    "trace_id": trace.trace_id,
    "agent_id": trace.agent_id,
    "error": str(e)
})
```

**Async Patterns**:
```python
# Use async/await for I/O operations
async def save_event(self, event: OpenCodeEvent) -> str:
    def _save():
        # SQLite operations in executor
        ...
    await asyncio.get_event_loop().run_in_executor(None, _save)

# Global store initialization pattern
event_store: EventStore = None

def init_router(store: EventStore):
    global event_store
    event_store = store
```

### TypeScript (Frontend)

**Imports** (in this order):
```typescript
// 1. React
import React, { useState, useEffect, useCallback } from 'react';

// 2. Third-party
import axios from 'axios';
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';

// 3. Local (use relative paths)
import { AgentTrace, Statistics } from '../types';
import { traceApi } from '../services/api';
import { useAgentStore } from '../stores/agentStore';
```

**Types**:
```typescript
// Use interface for object shapes
export interface AgentTrace {
  trace_id: string;
  session_id: string;
  status: 'idle' | 'pending' | 'running' | 'success' | 'failed';
}

// Use type for unions/utility types
type Status = 'online' | 'offline' | 'busy' | 'idle';
```

**Naming Conventions**:
- Components: `PascalCase.tsx` (e.g., `FlowCanvas.tsx`, `AgentList.tsx`)
- Functions/Variables: `camelCase` (e.g., `fetchAgents`, `handleClick`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `API_BASE`, `STATUS_COLORS`)
- API objects: `camelCase` + `Api` suffix (e.g., `traceApi`, `eventApi`)

**React Patterns**:
```typescript
// Functional components with explicit types
const AgentList: React.FC = () => {
  const { agents, loading, fetchAgents } = useAgentStore();
  
  useEffect(() => {
    fetchAgents();
  }, [fetchAgents]);
  
  return (<>...</>);
};

// dayjs with plugins
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.extend(relativeTime);
// Usage: dayjs(timestamp).fromNow()
```

## Project Structure

```
/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI app, startup/shutdown events
│   │   ├── config.py            # Pydantic settings
│   │   ├── websocket_manager.py # WebSocket connection manager
│   │   ├── api/
│   │   │   ├── __init__.py      # Router aggregation
│   │   │   ├── agents.py        # Agent execution endpoints
│   │   │   ├── traces.py        # Trace CRUD endpoints
│   │   │   └── events.py        # OpenCode event endpoints
│   │   ├── models/
│   │   │   ├── __init__.py      # Re-exports all models
│   │   │   ├── events.py        # OpenCodeEvent, EventCategory, EventType
│   │   │   └── agent_status.py  # AgentOnlineStatus, AgentStatus
│   │   ├── storage/
│   │   │   ├── trace_store.py   # SQLite trace storage
│   │   │   └── event_store.py   # SQLite event storage
│   │   └── hooks/
│   │       └── tracing_hooks.py # Agent lifecycle hooks
│   ├── tests/
│   │   ├── test_models.py
│   │   └── test_trace_store.py
│   ├── data/                    # SQLite database files
│   ├── requirements.txt
│   ├── requirements-dev.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.tsx              # Main app with navigation
│   │   ├── types/index.ts       # TypeScript interfaces
│   │   ├── services/api.ts      # Axios API client
│   │   ├── stores/agentStore.ts # Zustand state
│   │   ├── hooks/               # Custom React hooks
│   │   └── components/
│   │       ├── Canvas/          # ReactFlow visualization
│   │       ├── Trace/           # Trace timeline
│   │       ├── Monitor/         # Statistics panel
│   │       ├── Events/          # Event stream
│   │       └── Agents/          # Agent list
│   ├── package.json
│   └── Dockerfile
├── plugins/
│   └── opencode-viz-plugin/     # OpenCode integration plugin
├── docker-compose.yml
└── AGENTS.md
```

## Key Architecture Notes

### Backend Router Initialization Pattern
Routers use a global store that must be initialized at startup:
```python
# In main.py startup
from app.api.events import init_router as init_events_router
event_store = EventStore(database_url=settings.DATABASE_URL)
init_events_router(event_store)
```

### Database
- Development: SQLite (file: `backend/data/agent_viz.db`)
- Production: PostgreSQL with TimescaleDB extension
- Retention: 30 days (configurable via `TRACE_RETENTION_DAYS`)

### WebSocket
Real-time updates via WebSocket at `/api/v1/agents/ws`

## Git Commit Guidelines

- Use conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`
- Keep commits atomic and focused
- Example: `feat: add real-time trace visualization with WebSocket`

## Common Issues

1. **LSP errors in IDE**: Dependencies are in Docker, not locally installed. Errors are false positives.
2. **AgentStatusInfo vs AgentStatus**: The model class is `AgentStatus`, not `AgentStatusInfo`.
3. **dayjs fromNow()**: Requires `dayjs/plugin/relativeTime` to be imported and extended.
4. **Backend 404 on /events**: Ensure `init_events_router(event_store)` is called in main.py startup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cr330326)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cr330326)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
