## langchain-in-action

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

see [AGENTS.md](AGENTS.md)

## Project Overview

This is a **Multi-Agent Lab** project built with **LangChain + Ollama**. The project is structured as a learning-focused framework for building production-ready multi-agent systems with multiple business domains.

**Architecture**: Domain-driven multi-agent system with hierarchical agent structure
**Execution Patterns**:
- **Static Execution Plan**: Predefined sequential workflow using Skill Cards
- **Dynamic Agent**: LLM selects tools contextually based on user queries

**Current Status**: Multi-Agent Lab structure migration completed. LangGraph Supervisor pattern implemented with TodoAgent.

## Development Commands

### Running Examples

```bash
# Basic demos (learning progression)
uv run python -m src.examples.01_basic_agent        # Basic Agent
uv run python -m src.examples.02_file_agent         # File operations
uv run python -m src.examples.03_schedule_agent     # Schedule manager
uv run python -m src.examples.04_middleware_demo    # Middleware patterns

# Skill Card system
uv run python -m src.examples.05_skill_card_demo    # Skill Card demo
uv run python -m src.examples.07_executor_demo      # Executor demo

# Advanced integration (most important)
uv run python -m src.examples.08_real_tools_demo    # LLM + DB + Logic tools (verbose debugging)
uv run python -m src.examples.09_dynamic_agent      # Dynamic tool selection
uv run python -m src.examples.10_langgraph_supervisor  # LangGraph Multi-Agent Supervisor

# Infrastructure demos
uv run python -m src.examples.13_elasticsearch_demo # Elasticsearch integration
uv run python -m src.examples.14_redis_demo         # Redis integration

# Quick test script
uv run python quick_test.py
```

### Testing

```bash
# Run all tests
uv run pytest

# Run with verbose output
uv run pytest -v

# Run specific test file
uv run pytest tests/core/test_skill_card_manager.py

# Run specific test
uv run pytest tests/core/test_skill_card_manager.py::test_skill_card_load
```

### Code Quality

```bash
# Lint with ruff
uv run ruff check .

# Auto-fix issues
uv run ruff check --fix .

# Format code
uv run ruff format .
```

### Environment Setup

```bash
# Install dependencies
uv sync

# Check Ollama model availability
ollama list

# Start Ollama server (if not already running)
ollama serve

# Pull required model
ollama pull gpt-oss:20b
```

## Architecture

### Core Pattern: Skill Card System

The **Skill Card** is a JSON-based metadata pattern that defines Agent behavior:

```
User Query → SkillCardManager → SkillCardExecutor → Tools → Result
```

**Key Components:**

1. **SkillCard** (`src/multi_agent_lab/platform/skill_card/schema.py`): Pydantic schema defining agent metadata
2. **SkillCardManager** (`src/multi_agent_lab/platform/skill_card/manager.py`): Loads JSON files
3. **SkillCardExecutor** (`src/multi_agent_lab/platform/skill_card/executor.py`): Executes the plan

**Execution Flow:**

```python
# 1. Load Skill Card from JSON
manager = SkillCardManager()
card = manager.get("SC_SCHEDULE_001")

# 2. Register tools
executor = SkillCardExecutor(card, verbose=True)
executor.register_tool("parse_event_info", parse_event_info)
executor.register_tool("create_event", create_event)

# 3. Execute
result = executor.execute(
    user_query="내일 오후 2시에 팀 회의",
    context={"user_id": "user123"}
)
```

### Variable Substitution Pattern

Skill Cards use `${variable}` syntax for data flow between steps:

```json
{
  "step": 2,
  "action": "create_event",
  "input": {
    "title": "${event_data.title}",           // From step 1
    "start_time": "${free_slots.best_slot.start}"  // Nested access
  },
  "output_to": "created_event"
}
```

Implementation: `SkillCardExecutor._substitute_variables()` uses regex pattern `\$\{([^}]+)\}` to replace variables with actual values from `ExecutionContext.variables`.

### Tool Types

**1. LLM Tools** (Structured Output with Pydantic):
```python
from pydantic import BaseModel, Field

class EventInfo(BaseModel):
    title: str = Field(description="일정 제목")
    date: str = Field(description="날짜 (YYYY-MM-DD)")

@tool
def parse_event_info(query: str, verbose: bool = False) -> dict:
    llm = ChatOllama(model="gpt-oss:20b", temperature=0.0)
    structured_llm = llm.with_structured_output(EventInfo)
    result: EventInfo = structured_llm.invoke(prompt)
    return result.model_dump()
```

**2. DB Tools** (Database operations):
```python
@tool
def create_event(title: str, start_time: str) -> dict:
    event = {...}
    db.add_event(event)  # In-memory DB
    return {"success": True, "event": event}
```

**3. Logic Tools** (Business logic):
```python
@tool
def find_free_time(date: str, duration: int = 60) -> dict:
    # Complex scheduling logic
    busy_slots = get_busy_slots(date)
    available_slots = calculate_free_time(busy_slots)
    return {"available_slots": available_slots}
```

### Verbose Debugging System

The project uses a **two-level debugging** approach:

**Level 1: SkillCardExecutor verbose mode**
```python
executor = SkillCardExecutor(card, verbose=True)
# Prints step-by-step execution, variable substitution, timing
```

**Level 2: LangChain global debug**
```python
from langchain_core.globals import set_debug

if verbose:
    set_debug(True)  # Shows LLM prompts, responses, token counts
```

**Important**: Use `langchain_core.globals`, NOT `langchain.globals` (doesn't exist).

### Middleware System

Middleware pattern for pre/post processing:

```python
class BaseMiddleware:
    def pre_process(self, query: str) -> str:
        """Process before agent execution"""
        pass

    def post_process(self, result: str) -> str:
        """Process after agent execution"""
        pass
```

Current implementations:
- `PIIDetectionMiddleware`: Detects/masks phone, email, SSN
- `AuditLoggingMiddleware`: JSON Lines format logging

## Project Structure

```
src/multi_agent_lab/
├── core/                   # Framework core (Agent base classes, Middleware)
│   ├── agents/
│   │   └── base_agent.py
│   └── middleware/
│       ├── base.py
│       ├── pii_detection.py
│       └── audit_logging.py
│
├── platform/               # Execution platform
│   ├── langchain_adapter/
│   └── skill_card/         # ⭐ Skill Card pattern implementation
│       ├── schema.py       # Pydantic models
│       ├── manager.py      # Load JSON
│       └── executor.py     # Execute plan
│
├── domains/                # Business domains (independent agent systems)
│   ├── personal_assistant/
│   │   ├── agents/
│   │   │   └── schedule_manager.py  # Dynamic agent
│   │   ├── tools/
│   │   │   └── schedule_tools.py    # LLM/DB/Logic tools
│   │   ├── storage/
│   │   │   └── memory_db.py         # In-memory DB
│   │   └── skill_cards/
│   │       └── schedule_card.json   # Static plan definition
│   │
│   ├── financial/          # Financial domain (expandable)
│   │   ├── agents/
│   │   ├── tools/
│   │   └── storage/
│   │
│   └── research/           # Research domain (planned)
│
├── infra/                  # Infrastructure (external systems)
│   ├── llm/
│   ├── database/
│   │   ├── elasticsearch/
│   │   ├── redis/
│   │   └── postgres/
│   └── cache/
│
├── shared/                 # Shared components
│   ├── types/
│   ├── utils/
│   └── tools/              # Generic tools (calculator, get_current_time, etc.)
│
├── app/                    # Application composition (planned)
│   ├── personal_assistant_app.py
│   └── financial_app.py
│
└── interfaces/             # External interfaces (planned)
    ├── cli.py
    └── api.py

src/examples/               # Runnable demos
├── 01_basic_agent.py       # Basic Agent
├── 02_file_agent.py        # File operations Agent
├── 03_schedule_agent.py    # Schedule management Agent
├── 04_middleware_demo.py   # Middleware patterns
├── 05_skill_card_demo.py   # Skill Card demonstration
├── 07_executor_demo.py     # Executor demonstration
├── 08_real_tools_demo.py   # ⭐ Skill Card + Real Tools (primary integration demo)
├── 09_dynamic_agent.py     # Dynamic tool selection
├── 13_elasticsearch_demo.py # Elasticsearch integration
└── 14_redis_demo.py        # Redis integration
```

For detailed architecture documentation, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Key Concepts

### Static vs Dynamic Execution

| Aspect | Static Plan | Dynamic Agent |
|--------|-------------|---------------|
| Tool Selection | Predefined in JSON | LLM decides |
| Execution Order | Always same | Context-dependent |
| Predictability | High | Low |
| Flexibility | Low | High |
| Use Case | Compliance-critical | User experience |

**Static** (Step 04-05): `SkillCardExecutor` executes JSON execution_plan sequentially
**Dynamic** (Step 06): `ScheduleManagerAgent` uses LangChain's tool calling

See `docs/personal-assistant/patterns.md` for detailed comparison.

### Ollama Integration

**Model**: `gpt-oss:20b` (GPT-Oss-20B from Shallowmind)

**Setup:**
```bash
ollama pull gpt-oss:20b
ollama serve  # Must be running
```

**Usage:**
```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="gpt-oss:20b",
    temperature=0.0,  # Deterministic for parsing
)
```

**Troubleshooting**: If "Could not connect to Ollama" error, run `ollama serve` in separate terminal.

## Testing Patterns

### Test Structure

```python
# tests/conftest.py automatically sets working directory to project root
# No need for sys.path manipulation

def test_example():
    from multi_agent_lab.platform.skill_card import SkillCardManager

    manager = SkillCardManager()
    card = manager.get("SC_SCHEDULE_001")

    assert card is not None
    assert card.skill_id == "SC_SCHEDULE_001"
```

### Pytest Configuration

`pyproject.toml` sets:
```toml
[tool.pytest.ini_options]
pythonpath = ["src"]
testpaths = ["tests"]
```

This means imports work from `src/` root without relative paths.

## Common Patterns

### Creating a New Tool

```python
from langchain_core.tools import tool

@tool
def my_new_tool(param: str, verbose: bool = False) -> dict:
    """
    Tool description for LLM

    Args:
        param: Parameter description
        verbose: Enable debug output

    Returns:
        dict: Result with success/error keys
    """
    if verbose:
        print(f"[DEBUG] my_new_tool: {param}")

    try:
        result = do_something(param)
        return {"success": True, "data": result}
    except Exception as e:
        return {"success": False, "error": str(e)}
```

### Adding to Skill Card

```json
{
  "execution_plan": [
    {
      "step": N,
      "action": "my_new_tool",
      "input": {
        "param": "${previous_step.field}"
      },
      "output_to": "my_result",
      "on_error": "fail"  // or "skip"
    }
  ]
}
```

### Register in Executor

```python
from multi_agent_lab.domains.personal_assistant.tools.my_tools import my_new_tool

executor.register_tool("my_new_tool", my_new_tool)
```

## Documentation

Comprehensive docs in `docs/` organized hierarchically:

**Quick Start:**
1. `docs/README.md` - Navigation hub
2. `docs/personal-assistant/README.md` - Project intro
3. `docs/personal-assistant/concepts.md` - Core concepts (10 min read)

**Implementation:**
- `docs/personal-assistant/implementation-guide.md` - Tool creation, verbose debugging, best practices
- `docs/personal-assistant/patterns.md` - Static vs Dynamic detailed comparison
- `docs/personal-assistant/step-by-step/` - Step-by-step implementation guides

**General Learning:**
- `docs/learning-guide.md` - LangChain learning roadmap
- `docs/package-guide.md` - Python package structure guide

## Important Notes

### Import Paths

Always use absolute imports from `src/multi_agent_lab/`. The codebase has been migrated from the old flat structure to a domain-driven architecture.

```python
# ✅ Correct (current structure)
from multi_agent_lab.platform.skill_card import SkillCardManager
from multi_agent_lab.domains.personal_assistant.tools.schedule_tools import create_event
from multi_agent_lab.infra.database.elasticsearch import ElasticsearchClient
from multi_agent_lab.core.middleware import PIIDetectionMiddleware

# ❌ Wrong (old flat structure - no longer valid)
from core.skill_cards import SkillCardManager
from personal_assistant.tools import create_event

# ❌ Wrong (relative imports)
from ..platform.skill_card import SkillCardManager
```

**Important**: All paths in documentation referring to old structure (like `src/core/skill_cards/`) should be interpreted as `src/multi_agent_lab/platform/skill_card/`.

### LangChain Versions

This project uses **LangChain 1.0.5** split packages:
- `langchain-core`: Core abstractions
- `langchain-community`: Community integrations
- `langchain-ollama`: Ollama specific

**Note**: Some examples use `langchain_classic.agents` for compatibility with tool calling pattern.

### Database

Currently uses **in-memory database** (`src/multi_agent_lab/domains/personal_assistant/storage/memory_db.py`). Data doesn't persist between runs. This is intentional for demo purposes.

For production use, integrate with:
- **Elasticsearch**: `src/multi_agent_lab/infra/database/elasticsearch/` (already implemented)
- **Redis**: `src/multi_agent_lab/infra/database/redis/` (already implemented - caching and session management)
- **PostgreSQL**: Planned for relational data storage

### Korean Language

All user-facing messages and documentation are in Korean. LLM system prompts include `항상 한국어로 응답하세요.`

## Code Style and Conventions

From AGENTS.md - these are the repository guidelines:

### Module Organization
- `src/multi_agent_lab/core`: Agent primitives, skill cards (schema, manager, executor), middleware (audit + PII)
- `src/multi_agent_lab/platform`: Execution platform (LangChain adapter, Skill Card system)
- `src/multi_agent_lab/domains`: Business domains with runnable skills, tools, and storage
- `src/examples`: Narrative demos (keep inputs lightweight for fast execution)
- `tests`: Mirrors module tree structure
- `docs/` and `logs/`: Design notes and execution traces

### Naming Conventions
- **Python code**: snake_case for modules, functions, files; PascalCase for classes; UPPER_SNAKE_CASE for constants
- **Skill Card JSON**: lowerCamelCase for JSON keys (to match existing cards and executor expectations)
- **File naming**: Numbered examples use format `##_description.py` (e.g., `08_real_tools_demo.py`)

### Testing Strategy
- Use fixtures from `tests/conftest.py` (add new fixtures only when shared across multiple suites)
- Name tests: `test_<feature>_<scenario>`
- For Skill Card or tool changes: pair unit tests with `uv run python -m src.examples.08_real_tools_demo`
- Capture anomalies in `logs/` directory

### Commit Guidelines
- Follow conventional commit style: `docs: ...`, `feat: ...`, `fix: ...`
- Keep subject lines under 72 characters
- PR descriptions should include: motivation, solution outline, verification commands, and screenshots/JSON diffs for behavior changes
- Reference issues and flag breaking changes

### Agent & Skill Card Development
- Extend capabilities via `platform/skill_card/schema.py` or `executor.py` (not inside agents)
- Register new tools in domain-specific tool files (e.g., `domains/personal_assistant/tools/schedule_tools.py`)
- Document expected inputs/outputs in the matching Skill Card JSON
- Middleware is opt-in: subclass `BaseMiddleware` and pass through executor configuration

## Next Steps

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed roadmap.

**Immediate Next Steps:**
1. **Domain Supervisors**: Implement supervisor pattern for each domain
2. **Master Agent**: Create top-level orchestrator for multi-domain routing
3. **Message Bus**: Add event-driven communication between agents
4. **Additional Domains**: Expand financial and research domains

**Future Enhancements:**
- FastAPI interface for REST API
- Redis caching layer
- LangSmith integration for debugging
- Production deployment automation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sskim91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
