## dendrite

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

### Backend
```bash
# Start API server (primary mode)
python -m uvicorn api.server:app --host 0.0.0.0 --port 8000 --reload

# Terminal REPL mode (no web UI)
python main.py

# Install dependencies
pip install -e ".[api]"
```

### Frontend
```bash
cd frontend
npm run dev      # Dev server on :5173
npm run build    # Production build
```

## Environment

`.env` in project root:
```
OPENAI_API_KEY=<key>
OPENAI_BASE_URL=<base_url>   # e.g. https://jkwl.dmxapi.cn/v1
```

Model is configured in `config/user_RAG.json` under `llm.model`.

## Architecture

### Request Flow
```
Frontend (React :5173)
  → POST /api/canvas/{id}/execute
  → WebSocket /api/canvas/{id}/ws  (real-time events)
  → TaskPlanner.adecompose()       (LLM decomposes query into steps)
  → TaskExecutor.execute()         (runs steps in dependency order)
    → SimpleLLMClient.agenerate()  per step (stateless, no tools)
  → WebSocket events: execution_start → plan_ready → step_start → step_complete → task_complete
```

### Chat Flow
```
Frontend ChatPanel
  → GET /api/chat/stream?message=...  (SSE)
  → OrchestratorAgent.stream()
    → IntentRouter → AgentRole (RESEARCH / WRITING / CODE / fallback)
    → WorkerAgent with matching MCP servers
    → ChatOpenAI.stream_chat() (stateful, tool-calling)
```

### Key Components

| Component | Location | Role |
|-----------|----------|------|
| `OrchestratorAgent` | `agent/orchestrator.py` | Routes queries to worker agents; lazy-inits workers per role |
| `TaskPlanner` | `tasks/planner.py` | LLM decomposes user query into `Task` with ordered `TaskStep`s |
| `TaskExecutor` | `tasks/executor.py` | Runs steps concurrently (asyncio.gather) in topo order; uses `SimpleLLMClient` |
| `WorkerAgent` | `agent/agent.py` | Stateful agent with MCP tool-calling via `ChatOpenAI` |
| `SimpleLLMClient` | `agent/llm_client.py` | Stateless single-turn LLM calls (used by TaskExecutor) |
| `ChatOpenAI` | `agent/llm_client.py` | Stateful chat with tool-calling (used by WorkerAgent) |
| `RecipeStore` | `tasks/recipe_store.py` | Saves/matches past plans by embedding similarity |
| `KGStore` / `KGBuilder` | `graph/` | Knowledge graph extraction and storage |
| `ContextEngine` | `context_engine/` | Aggregates KG + RAG context for each query |

### Agent Roles & MCP Server Mapping
- `RESEARCH` → MCP servers with domain `SEARCH`
- `WRITING` → MCP servers with domains `PRODUCTIVITY`, `CODING`
- `CODE` → MCP servers with domain `CODING`
- fallback → servers selected by intent `tool_sets`

MCP server registry: `config/mcp_servers.json`

### Frontend State Machine
Canvas has two modes managed by `canvasStore.ts`:
- `kg_edit`: ReactFlow renders knowledge graph nodes; user submits queries
- `executing`: WebSocket-driven agent node animation; D3/framer-motion

WebSocket events consumed by frontend:
- `plan_ready` → render agent nodes with `node_id` positions
- `step_start` / `step_complete` / `step_error` → animate individual nodes
- `task_complete` → return to `kg_edit`, show result

### Data Persistence
All SQLite, stored in `data/`:
- `canvas.db` — canvas workspaces
- `memory.db` — working memory per task
- `workbench.db` — workbench items
- `knowledge.db` — knowledge graph nodes/edges

### System Prompts
Stored as markdown in `prompts/`. Loaded at runtime via `utils/prompt_loader.py`. Each worker role has its own file: `research_agent_system.md`, `writing_agent_system.md`, `code_agent_system.md`.

---
> Source: [Jiawe1Zhang/Dendrite](https://github.com/Jiawe1Zhang/Dendrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
