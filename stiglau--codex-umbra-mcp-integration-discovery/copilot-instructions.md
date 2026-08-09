## codex-umbra-mcp-integration-discovery

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Codex Umbra is an evolving chat interface system with four interconnected components:
- **The Oracle**: Mistral LLM via Ollama for natural language processing
- **The Visage**: React/TypeScript web interface (frontend)
- **The Conductor**: Python/FastAPI backend orchestrator
- **The Sentinel**: Python/FastAPI MCP server (internal system)

## Architecture

The system operates in three modes:
- **Architect Mode**: Learning and configuration phase
- **Running Mode**: Standard operational mode
- **In the Wild Mode**: Future autonomous operation

## Project Structure

```
├── docs/                    # Documentation organized by purpose
│   ├── architecture/        # System architecture documents
│   ├── components/          # Component-specific documentation
│   ├── planning/           # Project planning and guidelines
│   └── testing/            # Testing strategies and plans
├── scripts/                # Shell scripts for testing and deployment
├── tests/                  # Root-level test files and utilities
├── codex-umbra-visage/     # The Visage (React/TypeScript frontend)
├── conductor_project/      # The Conductor (Python/FastAPI backend)
└── mcp_server_project/     # The Sentinel (Python/FastAPI MCP server)
```

## Development Commands

### Quick Start All Components
```bash
./scripts/start_codex_umbra.sh    # Start all components
./scripts/docker-start.sh         # Start with Docker
```

### Individual Components

#### The Sentinel (MCP Server)
```bash
cd mcp_server_project
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
uvicorn mcp_server.main:mcp_app --reload --port 8001
```

#### The Conductor (Backend)
```bash
cd conductor_project
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

#### The Visage (Frontend)
```bash
cd codex-umbra-visage
npm install
npm run dev
```

#### The Oracle (Mistral LLM)
```bash
ollama pull mistral
ollama run mistral
```

## Testing

### End-to-End Testing
```bash
./scripts/test_full_integration.sh        # Comprehensive integration test
./scripts/test_integration_comprehensive.sh  # Extended integration test
./scripts/test_basic.sh                    # Basic functionality test
./scripts/test_quick_sanity.sh            # Quick sanity check
```

### Component Testing
```bash
./scripts/test_oracle_direct.sh           # Test Oracle directly
./scripts/test_oracle_integration.sh      # Test Oracle integration
./scripts/test_visage_v2_integration.sh   # Test Visage integration
```

### Python Components
```bash
pytest                    # Run all tests
pytest tests/unit/        # Unit tests only
pytest tests/integration/ # Integration tests only
pytest --cov             # With coverage
```

### Frontend
```bash
cd codex-umbra-visage
npm test                  # Unit tests
npm run test:e2e         # End-to-end tests
```

### Health Checks
```bash
curl http://localhost:8001/health  # Sentinel health
curl http://localhost:8000/health  # Conductor health
```

## Core Principles

**Prime Directive**: All code must be concise, efficient, and maintainable.

- Minimal comments unless essential for understanding
- Structured JSON responses between components
- FastAPI for Python backends with modular architecture
- React/TypeScript with component-centric structure
- Comprehensive testing at unit, integration, and E2E levels

## User Preferences

- **"Wrap it up"**: Commit all staged changes with appropriate commit message and branch creation
- **Project Organization**: Keep documentation organized in docs/ with purpose-based subfolders
- **Scripts & Tests**: Maintain scripts/ and tests/ folders for better organization
- **End-to-End Focus**: Prioritize working end-to-end functionality over individual components
- **Branch Management**: Create appropriately named branches for feature work when not on main

## Component Communication

1. User input → The Visage
2. The Visage → The Conductor (REST API)
3. The Conductor → The Oracle (Ollama API)
4. The Conductor → The Sentinel (REST API)
5. Response flow back through the chain

## Key Endpoints

### The Sentinel
- `GET /health` - Health check
- `GET /status` - System status

### The Conductor
- `POST /api/v1/chat` - Main chat endpoint
- `GET /health` - Health check

## Future Considerations

The system is designed for eventual self-modification capabilities. All architectural decisions support this long-term goal through modularity and clear separation of concerns.

---
> Source: [StigLau/codex-umbra_mcp-integration-discovery](https://github.com/StigLau/codex-umbra_mcp-integration-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
