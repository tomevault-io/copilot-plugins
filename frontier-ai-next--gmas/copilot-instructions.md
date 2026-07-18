## gmas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

When working with Python, invoke the relevant `/astral:<skill>` for `uv`, `ty`, and `ruff` to ensure best practices are followed.

## Development Commands

**Linting and Formatting:**

```bash
ruff check src/ tests/ examples/ benchmarks/
ruff format src/ tests/ examples/ benchmarks/
```

**Type Checking:**

```bash
ty check --ignore unresolved-import src/ tests/
```

**Testing:**

```bash
# Run all tests
pytest tests/ -v

# Single test file
pytest tests/path/to/test_file.py -v

# Single test function
pytest tests/path/to/test_file.py::test_function_name -v

# With coverage
coverage run -m pytest tests/ -v
coverage report --show-missing
```

**Multi-version Testing (via tox):**

```bash
tox                    # Runs py312, py313, lint, typecheck
tox -e py312          # Python 3.12 only
tox -e lint           # Lint and format check only
tox -e typecheck      # Type checking only
tox -e coverage       # Coverage report
```

## Architecture Overview

gMAS is a dynamic graph-based multi-agent system built on rustworkx. The architecture centers on three core abstractions:

### Core Abstractions

1. **RoleGraph** (`src/core/graph.py`) - The main directed graph structure
   - Stores agents as nodes with embeddings and hidden states
   - Maintains adjacency matrix (`A_com`) for fast access
   - Supports dynamic topology modifications at runtime
   - Converts to PyTorch Geometric format for GNN operations

2. **AgentProfile** (`src/core/agent.py`) - Individual agent definition
   - Each agent has local state (decentralized memory)
   - Stores embeddings and hidden states internally
   - Holds tool references and LLM configuration

3. **MACPRunner** (`src/execution/runner/`) - Execution engine
   - Modular design split across multiple files in `runner/` directory
   - `core.py`: Lifecycle, memory management, caller selection
   - `execution.py`: Simple and adaptive execution paths
   - `batch.py`: `run_round()` and `arun_round()` entrypoints
   - `stream.py`: Streaming API via `stream()` and `astream()`
   - `topology.py`: Dynamic topology changes
   - `state.py`: RunnerConfig, ExecutionContext, TopologyAction
   - `llm.py`: LLM caller protocols and factories
   - Supports structured prompts and per-agent LLM callers

### Key Differentiators from LangGraph

- **Dynamic topology**: Graph structure can change at runtime via `RoleGraph.update_communication()` or direct `rx.PyDiGraph` access
- **Decentralized memory**: Agent state is local to each `AgentProfile`, not centralized
- **Graph-first design**: Full access to adjacency matrices, edge attributes, node data

### Directory Structure

- `src/core/`: Graph, agent, schema, algorithms, GNN routing, metrics, visualization
- `src/execution/runner/`: Modular MACPRunner implementation
- `src/execution/scheduler.py`: Topological sort and SCC-based execution ordering
- `src/builder/`: GraphBuilder for constructing agent graphs
- `src/tools/`: BaseTool, ShellTool, WebSearchTool (modular with providers/fetchers), code interpreter
- `src/callbacks/`: Event system with manager and handlers (stdout, metrics, file)
- `src/config/settings.py`: FrameworkSettings using pydantic-settings (secure API key loading)

## Python Version

Minimum supported Python is **3.12**. Test matrix covers 3.12 and 3.13.

## Secure Configuration

Use `FrameworkSettings` for environment-based configuration. API keys support both direct values (`GMAS_API_KEY`) and file-based loading (`GMAS_API_KEY_FILE`). Invalid keys block startup without silent fallback.

---
> Source: [frontier-ai-next/gMAS](https://github.com/frontier-ai-next/gMAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
