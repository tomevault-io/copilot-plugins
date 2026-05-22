## agenticos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Environment Setup
```bash
# Install uv package manager (if not installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Setup development environment (creates .venv, installs dependencies)
cd agent-infra-docker
./scripts/dev_setup.sh
source .venv/bin/activate
```

### Code Quality
```bash
# Format code and fix imports (required before commits)
./agent-infra-docker/scripts/format.sh

# Type checking
mypy .

# Manual formatting commands
ruff format .
ruff check --select I --fix .
```

### Testing
```bash
# Full test suite (starts Docker containers automatically)
./agent-infra-docker/scripts/run_tests.sh

# Specific test types
./agent-infra-docker/scripts/run_tests.sh health
./agent-infra-docker/scripts/run_tests.sh agents
./agent-infra-docker/scripts/run_tests.sh fast

# Manual testing (requires running containers)
cd agent-infra-docker
docker compose up -d
pytest tests/ -v
docker compose down
```

### Infrastructure
```bash
# Start full development stack
cd agent-infra-docker
docker compose up -d --build

# Stop services
docker compose down

# Development server (alternative to Docker)
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### Package Management
```bash
# Use uv instead of pip for all package operations
uv pip install <package-name>

# After adding dependencies to pyproject.toml
./agent-infra-docker/scripts/generate_requirements.sh
```

## Architecture Overview

### Project Structure
- **Root (`agenticos`)**: Main Python package referencing the agent infrastructure
- **`agent-infra-docker/`**: Complete FastAPI application with Docker infrastructure
- **`agent-infra-docker/app/`**: Core application code (agents, database, main.py)
- **`agent-infra-docker/scripts/`**: Development automation scripts

### Core Architecture
This is an **Agno Framework**-based agent system providing a FastAPI web service for AI agents:

1. **AgentOS Pattern**: `app/main.py` creates an AgentOS instance that orchestrates multiple agents
2. **Agent Pattern**: Each agent (`app/agents/`) follows the pattern:
   ```python
   def get_agent_name(model_id: str, debug_mode: bool = False) -> Agent:
       return Agent(
           id="agent-id",
           model=ModelClass(id=model_id),
           tools=[Tools()],
           description=dedent("..."),
           instructions=dedent("..."),
           db=PostgresDb(...),  # Persistent storage
           knowledge=Knowledge(...)  # Optional RAG knowledge base
       )
   ```

3. **Database Integration**: PostgreSQL with pgvector extension for:
   - Agent conversation history (`db=PostgresDb`)
   - Vector embeddings for knowledge retrieval (`Knowledge` with `PgVector`)
   - Session management and user memory

4. **Pre-built Agents**:
   - **Web Search Agent**: DuckDuckGo integration with conversation memory
   - **Agno Assist**: RAG-enabled documentation assistant with knowledge base

### Key Dependencies
- **Agno Framework 2.0.4**: Core agent framework
- **FastAPI**: Web API layer
- **PostgreSQL + pgvector**: Database with vector capabilities
- **OpenAI/Google Gemini**: LLM providers (configurable per agent)
- **DuckDuckGo**: Web search tool

### Environment Configuration
Required environment variables:
```bash
OPENAI_API_KEY="your-api-key"     # Required for LLM access
DB_HOST=localhost                 # Database connection
DB_PORT=5432
DB_USER=ai
DB_PASS=ai
DB_DATABASE=ai
```

### Multi-Project Setup
The repository contains two related projects:
1. **Root project** (`pyproject.toml`): Meta-package depending on Agno
2. **Agent infrastructure** (`agent-infra-docker/pyproject.toml`): Complete application

When working on agents or the API, focus on the `agent-infra-docker/` directory.

### Testing Strategy
- **Integration tests**: `tests/` directory tests full agent functionality via API
- **Docker orchestration**: Tests automatically start/stop required services
- **Agent testing**: Tests both individual agent responses and conversation flow
- **Health checks**: Service availability and database connectivity

### Code Standards
- **Line length**: 120 characters (ruff configuration)
- **Type hints**: Required for all public functions (mypy enforced)
- **Import organization**: Automatic via ruff
- **Agent descriptions**: Use `textwrap.dedent()` for multi-line strings

---
> Source: [nexiouscaliver/AgenticOS](https://github.com/nexiouscaliver/AgenticOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
