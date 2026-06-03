## cmbagent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CMBAgent is a multi-agent system powered by AG2 (formerly AutoGen) for scientific discovery across scientific domains. The system follows a planning and control strategy with specialized agents for different research workflows.

The system uses a two-phase approach:
- **Planning**: A planner and plan reviewer design task execution strategies
- **Control**: Step-by-step execution where sub-tasks are handed to specialized agents

## Key Components

### Core Python Package (`cmbagent/`)
- **Main API**: `cmbagent.one_shot()` - primary function for task execution
- **Agent system**: Specialized agents in `agents/` directory, each with `.py` and `.yaml` configuration
- **Agent types**: Planning, control, coding (engineer, executor), research (researcher, summarizer), hypothesis generation, keyword extraction, and domain-specific templates
- **Context management**: Sophisticated context handling via `context.py` and `hand_offs.py`
- **Remote execution**: `execution/remote_executor.py` - delegates code execution to client

## Common Development Commands

### Python Package
```bash
# Install core only (lightweight)
pip install -e .

# Install with development tools
pip install -e ".[dev]"

# Install with local execution support (if not using remote execution)
pip install -e ".[local]"

# Install everything (all scientific packages)
pip install -e ".[all]"

# Run tests (individual Python scripts in tests/)
python tests/test_one_shot.py
python tests/test_engineer.py
```

## Architecture Details

### Agent System
- Each agent has a Python implementation and YAML configuration
- Agents are specialized for different workflows: engineering, research, planning, execution, hypothesis generation, keyword extraction
- Response formatters handle output formatting for specific use cases
- Controller manages workflow orchestration
- Domain-specific agents in `specialized/` serve as templates for users to add their own

### Multi-Modal Capabilities
- Plot generation capabilities
- File browser with inline image viewing

### Research Capabilities
- Domain-agnostic architecture supporting any scientific field
- Literature search and keyword extraction (`literature.py`, `aas_keyword_finder`)
- Example domain packages provided: materials science, biochemistry, astronomy, data science
- Users can easily add their own domain-specific dependencies in `pyproject.toml`

### Work Directory Structure
Tasks create organized output directories:
```
project_dir/
├── chats/          # Conversation history
├── codebase/       # Generated code
├── cost/           # Cost analysis
├── data/           # Generated data and plots
├── time/           # Timing reports
└── context/        # Context files (.pkl)
```

## Testing
- Tests are individual Python scripts in `tests/` directory
- Run specific test files directly: `python tests/test_<name>.py`
- Tests cover various agents and use cases (engineering, research, plotting, etc.)
- No centralized test runner - tests are designed as standalone demonstrations

## Configuration
- API keys required: `OPENAI_API_KEY`, optionally `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`
- Model configurations in `cmbagent/apis/` (JSON files for different providers)
- Agent configurations in individual `.yaml` files alongside Python implementations

## Key APIs
- `cmbagent.one_shot(task, agent='engineer', model='gpt-4o', work_dir=...)` - Main execution API
- Planning and control via specialized agent orchestration
- WebSocket streaming for real-time UI updates

---
> Source: [CMBAgents/cmbagent](https://github.com/CMBAgents/cmbagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
