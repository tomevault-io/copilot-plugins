## debate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-agent debate system built using the PydanticAI + LangGraph unified architecture. The system implements dialectical collaboration patterns where agents engage in structured debates to arrive at better solutions through thesis-antithesis-synthesis cycles.

## Commands

### Installation
```bash
# Navigate to the debate implementation
cd colab/debate

# Install dependencies
pip install -r requirements.txt
```

### Running Tests
```bash
# Run all tests with async support
pytest tests/ -v --asyncio-mode=auto

# Run specific test files
pytest tests/test_debate.py -v --asyncio-mode=auto
```

### Code Quality
```bash
# Format code with Black
black .

# Type checking with mypy
mypy agents/ graph/ --ignore-missing-imports

# Check formatting without making changes
black --check .
```

### Running the Application
```bash
# Start the FastAPI server
cd colab/debate
uvicorn api.main:app --reload --host 0.0.0.0 --port 8000

# For production
uvicorn api.main:app --host 0.0.0.0 --port 8000
```

### Validation Tests
```bash
# Test Archon connectivity
python -c "from archon.client import ArchonClient; client = ArchonClient(); print('✓ Archon connected')"

# Test PydanticAI agents
python -c "from agents.thesis_agent import thesis_agent; print('✓ Thesis agent loaded')"

# Test LangGraph workflow
python -c "from graph.workflow import get_debate_graph; print('✓ Graph compiled')"
```

## Architecture

### Unified System Architecture
This project uses ONE unified system where:
- **PydanticAI** creates intelligent, type-safe agents (the thinking)
- **LangGraph** orchestrates when/how agents execute (the coordination)
- **Archon MCP** provides the knowledge base (patterns and examples)
- **Web Search** provides fallback research when Archon lacks information

### Project Structure
```
debate/
├── colab/                      # Main collaborative system
│   ├── debate/                 # Dialectical debate implementation
│   │   ├── agents/            # PydanticAI agents
│   │   │   ├── thesis_agent.py      # Proposes solutions
│   │   │   ├── antithesis_agent.py  # Critiques proposals
│   │   │   ├── synthesis_agent.py   # Combines insights
│   │   │   └── moderator_agent.py   # Manages debate flow
│   │   ├── graph/             # LangGraph orchestration
│   │   │   ├── state.py       # Debate state management
│   │   │   ├── nodes.py       # Node wrappers for agents
│   │   │   └── workflow.py    # Debate workflow compilation
│   │   ├── api/               # FastAPI endpoints
│   │   └── tests/             # Test suite
│   └── PRPs/                  # Product Requirement Prompts
│       └── examples/          # Reference implementations
```

### Key Patterns

#### Agent-Node Integration
PydanticAI agents are wrapped in LangGraph nodes:
```python
async def thesis_node(state: DebateState, writer) -> dict:
    # Execute PydanticAI agent
    result = await thesis_agent.run(
        state["messages"][-1].content,
        deps=deps,
        message_history=state.get("pydantic_message_history", [])
    )
    # Return state update
    return {"agent_results": [result.data]}
```

#### Web Search Fallback Pattern
Agents automatically fall back to web search when Archon results are insufficient:
```python
# In agent tools
results = await ctx.deps.archon_client.perform_rag_query(query)

# Automatic fallback if enabled
if ctx.deps.enable_web_fallback:
    results = await search_web_fallback(
        query=query,
        archon_results=results,
        threshold=ctx.deps.web_search_threshold  # Default 0.6
    )
```

#### Dialectical Flow
1. **Thesis**: Agent proposes solution with high confidence
2. **Antithesis**: Agent critiques and finds flaws
3. **Synthesis**: Agent combines insights
4. **Moderation**: Agent tracks progress and terminates when converged

## Environment Setup

Create a `.env` file in `colab/debate/` (see `.env.example` for all options):
```bash
# Quick start - copy from .env.minimal.example
MODEL_PROVIDER=openai
OPENAI_API_KEY=sk-...
ARCHON_API_KEY=...

# Or see .env.example for full configuration options including:
# - Multiple LLM providers (OpenAI, Anthropic, Gemini)
# - Web search APIs (Google, Bing)
# - Performance tuning
# - Security settings
# - Monitoring configuration
```

## Development Guidelines

### Before Implementation
1. Query Archon MCP for existing patterns
2. Review examples in `PRPs/examples/`
3. Follow the unified architecture principles

### Web Search Fallback Configuration
Agents support automatic web search fallback when Archon results are insufficient:
- `enable_web_fallback`: Enable/disable web fallback (default: True)
- `web_search_threshold`: Minimum Archon relevance score (default: 0.6)
- Agents have both automatic fallback and explicit web search tools
- Web results are clearly marked with source URLs

### Testing Strategy
1. Unit test individual agents with TestModel
2. Test graph orchestration separately
3. Run integration tests for the complete system
4. Test web search fallback scenarios
5. Always use `pytest-asyncio` for async tests

### Code Style
- Use Black for formatting (line length 88)
- Type hints are required for all functions
- Follow PydanticAI patterns for agents
- Follow LangGraph patterns for orchestration

## API Endpoints

- `GET /` - Health check
- `POST /debate` - Start a new debate (returns streaming response)
- `GET /debate/{session_id}/status` - Get debate status
- `GET /debate/{session_id}/solution` - Get final solution

## Common Issues

1. **Import Errors**: Ensure you're in the correct directory (`colab/debate`)
2. **Async Warnings**: Use `--asyncio-mode=auto` with pytest
3. **Model Errors**: Check environment variables are set correctly
4. **Archon Connection**: Verify Archon MCP is accessible

## Important Notes

- This is ONE system, not two frameworks glued together
- Agents do the thinking, graphs do the orchestration
- Always query Archon before implementing new patterns
- Integration testing is crucial - test the unified system
- Stream responses for real-time user experience

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MikeSchirtzinger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MikeSchirtzinger)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
