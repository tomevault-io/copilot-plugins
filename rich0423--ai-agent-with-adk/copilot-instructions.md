## ai-agent-with-adk

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Google Cloud Run Day Workshop repository containing 4 workshop modules that demonstrate building and deploying AI agents using Google's Agent Development Kit (ADK) and Google Cloud services.

## Workshop Structure

- **workshop1/**: Basic AI agent with FastAPI and uvicorn
- **workshop2/**: Agent with Google Cloud Storage integration via MCP server
- **workshop3/**: Agent-to-Agent (A2A) communication demo with orchestrator pattern
- **workshop4/**: Advanced agent with LiteLLM integration and NLP capabilities

## Common Development Commands

### Running Services

Each workshop has different run commands:

**Workshop 1 (Basic Agent):**
```bash
cd workshop1
pip install -r requirements.txt
python main.py
# Or: uvicorn main:app --host 0.0.0.0 --port 8080
```

**Workshop 2 (GCS Agent Service):**
```bash
cd workshop2/news-gcs-agent-service
pip install -r requirements.txt
python main.py
```

**Workshop 2 (GCS MCP Server):**
```bash
cd workshop2/gcs-mcp-server
pip install -r requirements.txt
python main.py
```

**Workshop 3 (A2A Demo):**
```bash
cd workshop3/a2a-demo
pip install -e .  # Uses pyproject.toml

# Run remote agent:
uvicorn remote_agents.sentiment_analyzer_agent.agent:a2a_app --host 0.0.0.0 --port 8001

# Run orchestrator:
uvicorn orchestrator.__main__:app --host 0.0.0.0 --port 8000
```

**Workshop 4 (Advanced Agent):**
```bash
cd workshop4/news_agent_service
pip install -r requirements.txt
python main.py
```

### Docker Deployment

All workshops include Dockerfiles for containerization. Build and run containers:

```bash
# Example for any workshop with Dockerfile
docker build -t workshop-image .
docker run -p 8080:8080 workshop-image
```

## Architecture Patterns

### Google ADK Integration

All workshops use the Google Agent Development Kit (ADK) with a consistent pattern:

1. **Agent Definition**: Agents are defined using `google.adk.agents.Agent` class
2. **FastAPI Integration**: `google.adk.cli.fast_api.get_fast_api_app()` creates the web interface
3. **Tool Functions**: Custom tools are defined as Python functions and passed to the agent
4. **Web Interface**: Most services enable web UI with `web=True` parameter

### Service Architecture

- **Workshop 1-2, 4**: Single agent services with FastAPI
- **Workshop 3**: Distributed architecture with orchestrator + remote agents
  - Orchestrator coordinates multiple specialized agents
  - Remote agents run independently and communicate via HTTP

### Environment Configuration

- Services use `PORT` environment variable (defaults to 8080/8000)
- Workshop 3 uses dotenv for configuration management
- All services bind to `0.0.0.0` for Cloud Run compatibility

## Dependencies

### Core Dependencies
- `google-adk`: Google Agent Development Kit (version pinned to 1.13 in workshop1/4)
- `fastapi`: Web framework
- `uvicorn`: ASGI server

### Workshop-Specific Dependencies
- `google-cloud-storage`: GCS integration (workshop 2)
- `fastmcp`: MCP server functionality (workshop 2)
- `litellm`: LLM abstraction layer (workshop 4)
- `nltk`: Natural language processing (workshop 4)
- `python-dotenv`: Environment management (workshops 2, 3, 4)

## Development Notes

- All main.py files follow similar FastAPI + ADK pattern
- Agent definitions are in `*_agent/agent.py` files
- Workshop 3 uses pyproject.toml instead of requirements.txt
- Cloud Run deployment ready with PORT environment variable handling

---
> Source: [RICH0423/ai-agent-with-adk](https://github.com/RICH0423/ai-agent-with-adk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
