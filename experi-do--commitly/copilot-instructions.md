## commitly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Commitly** is an AI-powered, multi-agent commit automation system that orchestrates the entire post-commit workflow through a local Python pipeline. It uses LangGraph to coordinate seven specialized agents that handle code validation, testing, refactoring, synchronization, Slack integration, and reporting.

**Key Value**: Automates the entire process after Git commits through a local multi-agent pipeline with minimal manual intervention (single approval gate via SyncAgent).

## Development Commands

### Setup & Environment

```bash
# Install dependencies
poetry install

# Load environment variables (required before running commands)
set -a && source .env && set +a

# Initialize project structure (.commitly directory)
poetry run commitly init
```

### Running the Pipeline

```bash
# Execute the full pipeline after making changes
poetry run commitly commit -m "Your commit message"

# Equivalent to:
poetry run commitly git commit -m "Your commit message"
```

### Code Quality & Testing

```bash
# Run linting (Ruff)
ruff check src/

# Format code (Black)
black src/

# Type checking (MyPy)
mypy src/commitly/

# Run tests
pytest tests/

# Run tests with coverage
pytest --cov=src/commitly tests/

# Run a single test file
pytest tests/test_agent.py

# Run a specific test
pytest tests/test_agent.py::test_clone_agent_execution
```

### Debugging & Inspection

```bash
# Check pipeline status
poetry run commitly status

# Generate a report from pipeline execution
poetry run commitly report --from 2025-10-20 --to 2025-10-22

# View logs from a specific agent
cat .commitly/logs/{agent_name}/$(ls -t .commitly/logs/{agent_name} | head -1)

# Check cached output from previous agent runs
cat .commitly/cache/{agent_name}.json
```

## Architecture Overview

### Core Pattern: LangGraph-Based Orchestration

The pipeline follows a **sequential agent orchestration pattern** where a central `CommitlyPipeline` class (in `src/commitly/pipeline/graph.py`) constructs a LangGraph StateGraph with seven nodes. Each agent executes sequentially, sharing state through a `RunContext` (TypedDict) that persists data between agents.

```
User Workspace
    ↓ (git commit)
[CommitlyPipeline] orchestration layer
    ↓
[CloneAgent]       → Hub setup & local change sync
    ↓
[CodeAgent]        → Static analysis, SQL extraction
    ↓
[TestAgent]        → Test execution, SQL optimization
    ↓
[RefactoringAgent] → Code quality improvement
    ↓
[SyncAgent]        → USER APPROVAL GATE → Remote push
    ↓
[SlackAgent]       → Slack integration (non-blocking)
    ↓
[ReportAgent]      → Report generation (non-blocking)
```

### Key Design Principles

1. **Template Method Pattern**: `BaseAgent` (src/commitly/agents/base.py) defines the `run()` wrapper that handles error handling, output caching, and persistence. Each agent implements `execute()` for custom logic.

2. **Hub Repository Pattern**: Agents work in a shared hub directory (cloned from remote) rather than modifying the user's workspace directly. This provides safety and auditability. Only SyncAgent can push changes back to the user's workspace.

3. **Approval Gate**: SyncAgent is the only point requiring user interaction. It summarizes all changes and requires explicit approval before pushing to remote.

4. **Error Handling & Rollback**: Failed agents trigger rollback logic that deletes created branches and resets the hub to the last successful state. Non-blocking agents (Slack, Report) don't abort the pipeline if they fail.

5. **Comprehensive Logging**: Every agent logs to `.commitly/logs/{agent_name}/{timestamp}.log`. Outputs are also cached as JSON in `.commitly/cache/{agent_name}.json` for traceability.

### File Organization

```
src/commitly/
├── cli/                 # Command-line interface entry point
├── core/                # Shared utilities (config, git, logging, LLM)
├── pipeline/            # LangGraph orchestration (CommitlyPipeline)
├── agents/              # Seven agent implementations
│   ├── base.py          # BaseAgent abstract class (critical)
│   ├── clone/           # Hub sync
│   ├── code/            # Validation & SQL parsing
│   ├── test/            # Testing & SQL optimization
│   ├── refactoring/     # Code improvement
│   ├── sync/            # Remote push & approval gate
│   ├── slack/           # Slack integration
│   └── report/          # Report generation
└── tests/               # Test suite (placeholder)
```

## Key Files & Classes

### Pipeline Orchestration
- **`src/commitly/pipeline/graph.py:CommitlyPipeline`** - Main orchestrator that builds the LangGraph and executes agents sequentially. Initializes shared config, LLM client, Git manager, and RunContext.

### Base Agent Framework
- **`src/commitly/agents/base.py:BaseAgent`** - Abstract base class defining the agent execution template. All agents inherit from this. Key methods: `run()` (executes with error handling), `execute()` (override in subclasses), `_save_output()` (persists to cache).

### Agent Implementations
- **`src/commitly/agents/clone/agent.py:CloneAgent`** - Creates hub repo, syncs with remote, applies local changes
- **`src/commitly/agents/code/agent.py:CodeAgent`** - Runs static checks (ruff, mypy), executes user application, extracts SQL queries
- **`src/commitly/agents/test/agent.py:TestAgent`** - Optimizes SQL queries (LLM generates candidates, EXPLAIN ANALYZE evaluates), runs test suite
- **`src/commitly/agents/refactoring/agent.py:RefactoringAgent`** - Removes duplicate code, adds error handling via LLM, applies formatting
- **`src/commitly/agents/sync/agent.py:SyncAgent`** - THE APPROVAL GATE: summarizes changes, waits for user confirmation, pushes to remote
- **`src/commitly/agents/slack/agent.py:SlackAgent`** - Queries Slack, matches messages, auto-replies (non-blocking)
- **`src/commitly/agents/report/agent.py:ReportAgent`** - Generates markdown/PDF/HTML reports from logs (non-blocking)

### Core Infrastructure
- **`src/commitly/core/config.py`** - YAML configuration loader with environment variable substitution (`${VAR_NAME}`)
- **`src/commitly/core/context.py`** - TypedDict definitions for RunContext (shared state) and AgentOutput (standardized results)
- **`src/commitly/core/git_manager.py`** - GitPython wrapper for branch/commit operations
- **`src/commitly/core/llm_client.py`** - OpenAI API wrapper
- **`src/commitly/core/logger.py`** - Per-agent logging to `.commitly/logs/{agent_name}/`
- **`src/commitly/core/rollback.py`** - Failure handling, branch deletion, hub cleanup

### Special Components
- **`src/commitly/agents/code/sql_parser.py`** - Parses SQL queries from code files
- **`src/commitly/agents/test/sql_optimizer.py`** - LLM-based SQL optimization with EXPLAIN ANALYZE evaluation

## Configuration

### config.yaml Locations

1. **Project-level**: `config.yaml` in repo root (version controlled)
2. **Runtime**: `.commitly/config.yaml` (created by `commitly init`, can be project-specific)

### Essential Config Sections

```yaml
llm:
  enabled: true
  model: gpt-4o-mini
  api_key: ${OPENAI_API_KEY}

execution:
  command: python main.py          # User app to run
  timeout: 300

test:
  timeout: 300

pipeline:
  cleanup_hub_on_failure: false    # Control hub cleanup on errors

database:                           # For SQL optimization
  host: ${DB_HOST}
  port: 5432
  user: ${DB_USER}
  password: ${DB_PASSWORD}
  dbname: ${DB_NAME}
```

### Environment Variables

`.env` file must define (sourced before running pipeline):
- `OPENAI_API_KEY` - Required for LLM operations
- `DB_USER`, `DB_PASSWORD`, `DB_NAME` - For SQL optimization
- Any custom variables referenced in config.yaml via `${VAR_NAME}`

## Common Development Tasks

### Adding a New Agent

1. Create `src/commitly/agents/{agent_name}/agent.py`
2. Inherit from `BaseAgent`
3. Implement `execute(self, context: RunContext) -> Dict[str, Any]`
4. Return structured data matching your agent's requirements
5. Add agent import and node to LangGraph in `src/commitly/pipeline/graph.py:build_graph()`
6. Create corresponding log directory in `.commitly/logs/{agent_name}/`

### Debugging Agent Failures

1. Check the agent's log: `.commitly/logs/{agent_name}/{latest_timestamp}.log`
2. Inspect cached output: `.commitly/cache/{agent_name}.json` (shows status, error details, and previous results)
3. Review hub state: `.commitly/hub/` contains the state agents modified
4. Use `git log` in hub to trace branch history
5. If needed, manually reset hub: `git -C .commitly/hub reset --hard {branch}`

### Modifying Agent Behavior

All agent logic is in their respective `execute()` methods. Changes here are immediate:
- **Clone/Sync logic**: `agents/clone/agent.py`, `agents/sync/agent.py`
- **Code checks**: `agents/code/agent.py`, `agents/code/static_checker.py`
- **Test optimization**: `agents/test/agent.py`, `agents/test/sql_optimizer.py`
- **Refactoring rules**: `agents/refactoring/agent.py` (edit the refactoring prompts)

### Adding Tests

Place test files in `tests/` with `test_*.py` naming. Use pytest fixtures to mock:
- `RunContext` (create a dict matching the TypedDict schema)
- `LLMClient` (mock OpenAI responses)
- `GitManager` (use temporary git repos or mocks)

Example:
```python
def test_clone_agent_execution(tmp_path):
    context = {
        "pipeline_id": "test-123",
        "hub_path": str(tmp_path / "hub"),
        # ... other required fields
    }
    agent = CloneAgent()
    result = agent.run(context)
    assert result["status"] == "success"
```

## Testing with Coverage

```bash
# Run all tests with coverage report
pytest --cov=src/commitly --cov-report=html tests/

# Open coverage report
open htmlcov/index.html

# Check coverage for specific file
pytest --cov=src/commitly/agents/base --cov-report=term-missing tests/
```

## Performance Considerations

1. **LLM Calls**: Each refactoring operation triggers 2 LLM calls per file (duplication removal, error handling addition). Monitor token usage.
2. **SQL Optimization**: TestAgent generates 3 SQL candidates and runs EXPLAIN ANALYZE on each. Ensure database is accessible.
3. **Hub Operations**: Clone creates a shallow clone by default. For large repos, clone can be slow.
4. **Slack Queries**: SlackAgent queries Slack API for messages in a time range. Large time ranges may hit rate limits.

## Troubleshooting

| Issue | Diagnosis | Solution |
|-------|-----------|----------|
| `OPENAI_API_KEY not found` | Missing `.env` or not sourced | Run `set -a && source .env && set +a` before pipeline |
| Agent hangs | Timeout configuration | Increase `execution.timeout` or `test.timeout` in config.yaml |
| Hub sync fails | Network/permission issue | Check `.commitly/hub/.git/config` for remote URL |
| Test optimization fails | DB not accessible | Verify `database.*` config matches your PostgreSQL setup |
| Refactoring introduces bugs | LLM output quality issue | Review model choice or refactoring prompts in agent code |

## Language & Dependencies

- **Python**: 3.11+ (specified in pyproject.toml)
- **Package Manager**: Poetry (all dependencies in pyproject.toml)
- **Key Frameworks**: LangGraph (orchestration), LangChain (LLM abstraction), GitPython (git ops), OpenAI (LLM provider)
- **Code Quality**: Ruff (linting, format checking), Black (formatter), MyPy (type checker)
- **Testing**: Pytest, Pytest-cov

## Important Code Style Rules

From `pyproject.toml`:
- **Line length**: 100 characters
- **Python version**: 3.11+
- **Ruff lint rules**: E, F, I, N, W, UP (imports, naming, warnings, upgrades)
- **Type checking**: Strict MyPy mode with `disallow_untyped_defs = true`

All functions must have type annotations. Use TypedDict for structured data (as done in `core/context.py`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/experi-do)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/experi-do)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
