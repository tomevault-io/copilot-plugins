## incident-responder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Intelligent DevOps Multi-Agent System** that automatically investigates production incidents using CrewAI. It provides a REST API that triggers a 3-agent system (Log Analyst, Code Historian, Incident Commander) to analyze logs, search git history, and generate post-mortem reports.

## Common Commands

### Running the Application
```bash
python main.py
# or
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### Testing
```bash
pytest                                    # All tests
pytest tests/unit -v                      # Unit tests only
pytest tests/integration -v               # Integration tests only
pytest tests/e2e -v -m slow --timeout=300 # E2E tests only
pytest -k "test_log_parser"               # Pattern matching
```

### Code Quality
```bash
make format    # Format with ruff
make lint      # Lint with ruff
make check     # Format + lint check
```

### Using Make
```bash
make install        # Install deps (uv sync)
make install-dev    # Install all deps including dev/test
make test           # Run all tests with coverage
make test-unit      # Unit tests only
make test-integration  # Integration tests only
make test-e2e       # E2E tests only
make clean          # Remove cache files
```

## Architecture

### The 3-Agent System

1. **Log Analyst** - Parses logs using regex to identify errors, stack traces, and anomalies
2. **Code Historian** - Searches git commits for recent changes correlating with incident timeline
3. **Incident Commander** - Synthesizes findings into post-mortem reports with RCA and recommendations

### Agent/Task Configuration

- Agents are defined in `src/incident_responder/config/agents.yaml`
- Tasks are defined in `src/incident_responder/config/tasks.yaml`
- Crew orchestration in `src/incident_responder/crew.py` uses `@agent` and `@task` decorators

### Key Source Files

| File | Purpose |
|------|---------|
| `src/incident_responder/crew.py` | CrewAI orchestration with agent/task decorators |
| `src/incident_responder/api/routes.py` | FastAPI endpoints |
| `src/incident_responder/tools/log_parser_tool.py` | Regex-based log parser |
| `src/incident_responder/tools/git_search_tool.py` | Git commit search |
| `src/incident_responder/utils/config.py` | Configuration singleton |
| `main.py` | Application entry point |

## Configuration

Environment variables (in `.env`):
- `LLM_PROVIDER` - LLM provider: "azure" or "ollama" (default: "azure")
- `OLLAMA_MODEL` - Ollama model (default: `qwen3:14b`)
- `OLLAMA_BASE_URL` - Ollama server (default: `http://localhost:11434`)
- `OLLAMA_API_KEY` - Optional API key for Ollama Cloud
- `LOG_DIRECTORY` - Path to log files (default: `./data/logs`)
- `REPORTS_DIRECTORY` - Path for output reports (default: `./reports`)
- `GIT_REPO_PATH` - Path to git repository (default: `./data/mock_repo`)

To use Ollama, set `LLM_PROVIDER=ollama` in `.env` and ensure Ollama is running.

## API Endpoints

- `POST /api/v1/trigger-investigation` - Start an incident investigation
- `GET /api/v1/investigation/{id}` - Check investigation status
- `GET /api/v1/health` - Health check
- API docs at `http://localhost:8000/docs`

## Adding New Agents/Tools

### Adding a New Tool
1. Create tool class in `src/incident_responder/tools/` extending `BaseTool`
2. Define Pydantic input schema
3. Register in `tools/__init__.py`

### Adding a New Agent
1. Define agent in `config/agents.yaml` with role, goal, backstory
2. Add agent method in `crew.py` with `@agent` decorator
3. Create task in `config/tasks.yaml`
4. Add task method in `crew.py` with `@task` decorator

## Test Validation

Reports must contain these keywords for validation:
- "Error" - describing the issue
- "Commit" - referencing code changes
- "Recommendation" - prevention steps

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mukunduashok)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mukunduashok)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
