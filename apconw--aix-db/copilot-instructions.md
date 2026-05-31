## aix-db

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plan Mode

When in plan mode, always write plan files to the project directory `.claude/plans/` instead of the default `~/.claude/plans/`. Use descriptive filenames based on the task topic (e.g., `common-agent-refactor-skill-mcp.md`).

## Build & Run Commands

```bash
# Backend - start dev server (Sanic on port 8088)
python serv.py

# Backend - install dependencies
uv add <package> --frozen    # add new dependency without upgrading existing ones
uv pip freeze > requirements.txt  # regenerate lockfile after changes

# Frontend - dev server
cd web && pnpm install && pnpm dev

# Docker
make up          # start services (docker-compose)
make down        # stop services
make logs        # view logs
make build       # build app image (~30s, requires base image)
make build-base  # rebuild base image (only when deps change)

# Code formatting
black .          # line-length=88, target py311

# Tests (ad-hoc, organized by feature under tests/)
python -m pytest tests/<subdir>/<test_file>.py
```

## Architecture

**Aix-DB** is an LLM-powered data analysis platform (ChatBI). Users ask questions in natural language; the system generates SQL, executes it, and returns visualized results.

### Backend (Python 3.11, Sanic)

```
serv.py                          # Entry point - Sanic app, autodiscovers controllers
├── controllers/                 # REST API blueprints (auto-registered via autodiscover)
│   ├── llm_chat_api.py         # Main chat endpoint: POST /dify/get_answer (SSE streaming)
│   ├── db_chat_api.py          # Database Q&A endpoints
│   ├── skill_api.py            # GET /system/skill/list
│   └── datasource_api.py       # Datasource CRUD
├── services/
│   └── llm_service.py          # Routes requests by qa_type to appropriate agent
├── agent/                       # Four independent agent systems
│   ├── common_react_agent.py   # COMMON_QA - general chat (LangChain + MCP)
│   ├── text2sql/               # DATABASE_QA - Text-to-SQL (LangGraph StateGraph)
│   ├── excel/                  # FILEDATA_QA - Excel analysis (DuckDB)
│   └── deepagent/              # REPORT_QA - deep research (deepagents library)
│       ├── AGENTS.md           # Agent instructions (loaded as memory)
│       ├── skills/             # SKILL.md instruction documents
│       └── tools/              # SQL tools + ToolCallManager (loop prevention)
├── model/                       # SQLAlchemy models + Pydantic schemas
├── common/                      # Utilities (LLM, MinIO, crypto, datasource connections)
└── config/                      # Environment loading, logging config
```

### Request Flow (Chat)

```
POST /dify/get_answer { query, qa_type, chat_id, datasource_id, ... }
  → llm_service.LLMRequest.exec_query()
    → qa_type routing:
      COMMON_QA   → CommonReactAgent.run_agent()
      DATABASE_QA → Text2SqlAgent.run_agent()
      FILEDATA_QA → ExcelAgent.run_excel_agent()
      REPORT_QA   → DeepAgent.run_agent()
  → SSE streaming response: data:{data:{messageType,content},dataType}\n\n
```

### Agent Systems (independent, do not share code between them)

- **Text2SqlAgent**: LangGraph StateGraph pipeline (datasource_selector → schema_inspector → sql_generator → permission_filter → sql_executor → chart_generator → summarizer)
- **DeepAgent**: `deepagents.create_deep_agent()` with skills, multi-phase tracking (PLANNING→EXECUTION→SUB_AGENT→REPORTING), `<details>` HTML for thinking visibility
- **CommonReactAgent**: `langchain.agents.create_agent()` with MCP tools via `MultiServerMCPClient`
- **ExcelAgent**: Parses Excel to DuckDB, then SQL analysis pipeline

### Frontend (Vue 3 + TypeScript + Vite)

```
web/src/
├── views/chat/index.vue        # Main chat interface (~3700 lines)
├── views/skill-center.vue      # Skill browsing page
├── api/index.ts                # Chat API (SSE fetch to /sanic/dify/get_answer)
├── store/business/index.ts     # Pinia store (qa_type, file_list, task_id)
└── components/MarkdownPreview/ # Renders markdown + HTML (including <details>)
```

### Key Patterns

- **SSE format**: All agents stream responses as `data:{"data":{"messageType":"continue","content":"..."},"dataType":"t02"}\n\n`
- **dataType values**: `t02` (text answer), `t04` (business/chart data), `t09` (stream end)
- **Datasource support**: MySQL, PostgreSQL, Oracle, SQL Server, ClickHouse, DM, Doris, StarRocks (via SQLAlchemy or native drivers)
- **MCP integration**: External mcp-hub service, configured via `MCP_HUB_COMMON_QA_GROUP_URL` env var
- **Skills**: Markdown instruction documents (`SKILL.md` with YAML frontmatter) loaded as LLM context, not executable tools
- **Auth**: JWT tokens in `Authorization: Bearer <token>` header, decoded via `services/user_service.decode_jwt_token()`

### Environment

Key env vars (see `.env.dev` and `docker/docker-compose.yaml`):
- `DATABASE_URL` - PostgreSQL connection for app metadata
- `LLM_MODEL_NAME`, `LLM_API_KEY`, `LLM_API_BASE` - LLM configuration
- `MINIO_*` - File storage
- `MCP_HUB_COMMON_QA_GROUP_URL` - MCP tool hub endpoint
- `LANGFUSE_TRACING_ENABLED` - Enable observability tracing

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `python3 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"` to keep the graph current

---
> Source: [apconw/Aix-DB](https://github.com/apconw/Aix-DB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
