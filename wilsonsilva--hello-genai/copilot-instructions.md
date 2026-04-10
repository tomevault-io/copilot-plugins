## hello-genai

> This file provides guidance to AI agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

This is a three-tier Python framework for building AI agents with Google's Gemini API. The architecture separates reusable infrastructure (ADK - Agent Development Kit), domain-agnostic tools, and specialized agent implementations, demonstrating modular design for conversational AI with function calling capabilities.

**Key Components**:
- **ADK (Agent Development Kit)**: Core framework in `adk/` - provides base Agent class and configuration management
- **Tools**: Reusable functions in `tools/` - file operations, future extensions for web, data, etc.
- **Utils**: Utility functions in `utils/` - decorators and helper functions
- **Agents**: Specialized implementations in `agents/` - currently FileAgent, designed for easy extension

For detailed architecture, data flow diagrams, module-by-module documentation, and extension patterns, see [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md).

## Environment Setup

**Package Manager**: This project uses `uv` (not pip or poetry).

**Python Version**: 3.14 (managed via `.python-version`)

**Environment Variables**: Copy `.env.example` to `.env` and add your Gemini API key from https://aistudio.google.com/api-keys

```bash
cp .env.example .env
# Then edit .env to add your GEMINI_API_KEY
```

## Development Commands

### Setup and Installation
```bash
# Install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate
```

### Running the Application
```bash
# Run the main script
uv run main.py

# Or with activated venv
python main.py
```

### Adding Dependencies
```bash
# Add a new dependency
uv add <package-name>

# Add a dev dependency
uv add --dev <package-name>
```

### Code Quality Tools
```bash
# Format code with black
uv run black .

# Lint code with ruff
uv run ruff check .

# Auto-fix linting issues
uv run ruff check --fix .
```

### Testing and Coverage
```bash
# Run tests
uv run pytest

# Run tests with coverage
uv run coverage run -m pytest

# View coverage report in terminal
uv run coverage report

# Generate HTML coverage report and view in browser
uv run coverage html
open coverage/html/index.html

# Generate all report formats (lcov, HTML, XML)
uv run coverage lcov && uv run coverage html && uv run coverage xml
```

**Coverage Configuration**:
- Coverage is configured in `.coveragerc`
- Tracks coverage for `adk/`, `agents/`, `tools/`, and `utils/` modules
- Reports are generated in multiple formats:
  - Terminal output with missing lines
  - LCOV: `reports/lcov.info` (used by qlty)
  - XML: `coverage/coverage.xml`
  - HTML: `coverage/html/index.html` (for local viewing)
- GitHub CI automatically uploads coverage to qlty on every push/PR

### Running Examples
```bash
# Run the main script (demonstrates basic agent with function calling)
uv run main.py

# Run comprehensive examples with multiple tools
uv run example_with_tools.py
```

## Code Architecture

### Three-Tier Design

1. **ADK (Agent Development Kit)** - `adk/`
   - Core framework providing base `Agent` class and `AgentConfiguration`
   - Model-agnostic, tool-agnostic foundation
   - No domain knowledge or specific use cases

2. **Utils** - `utils/`
   - Utility functions and decorators (currently: `log_function_call` decorator)
   - Reusable across the entire project
   - Framework-level support utilities

3. **Tools** - `tools/`
   - Reusable functions (currently: file operations)
   - Type-hinted with descriptive docstrings
   - Modular and shareable across agents

4. **Agents** - `agents/`
   - Specialized implementations (currently: `FileAgent`)
   - Pre-configured with specific tools and system instructions
   - Each can have its own `.env` configuration

### Agent Class (`adk/agent.py`)
The core `Agent` class provides:
- **Conversation History**: Maintains context in `self.contents` as a list of message dictionaries
- **Function Calling**: Supports tool execution via Google GenAI's native function calling
- **Recursive Tool Execution**: When the model requests a function call, the agent executes it and automatically sends results back
- **Configuration Management**: Uses singleton `agent_configuration` for defaults

### Function Calling Flow
1. User message sent with tool declarations to the model
2. Model responds with text and/or function call requests
3. If function calls exist, they are executed locally and logged
4. Function results are sent back to the model (via recursive `run()` call)
5. Model generates final natural language response

### Tool Implementation (`file_tools.py`)
Tools are standard Python functions with:
- Type hints for parameters
- Docstrings describing functionality and arguments (used by the model)
- Optional `@log_function_call` decorator from `utils/` for debugging

The model automatically converts docstrings into function schemas for the API.

### Message Structure
Messages follow this format:
```python
{"role": "user", "parts": [{"text": "user message"}]}
```

Model responses are appended directly from `response.candidates[0].content`

## Important Notes

- The project uses **Gemini 2.5 Flash** model by default (`gemini-2.5-flash`)
- API key must be set as `GEMINI_API_KEY` environment variable
- The Agent class maintains conversational context automatically across multiple `run()` calls
- Function calls are logged to stdout with `[Function Call]` prefix when using the `@log_function_call` decorator from `utils/`
- Test suite exists with tests for tools, decorators, and other components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wilsonsilva)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wilsonsilva)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
