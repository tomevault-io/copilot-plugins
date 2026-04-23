## personalutils

> > **For AI Assistants**: This document provides comprehensive guidance for understanding and working with the personalUtils codebase.

# CLAUDE.md - AI Assistant Guide

> **For AI Assistants**: This document provides comprehensive guidance for understanding and working with the personalUtils codebase.

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Key Concepts](#key-concepts)
- [Development Workflows](#development-workflows)
- [Code Conventions](#code-conventions)
- [Testing](#testing)
- [Configuration](#configuration)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

---

## Overview

**personalUtils** is a Python-based personal utility toolkit featuring:

1. **ChatSystem** - Production-ready GPT-powered chat system with function calling and streaming
2. **Four Specialized AI Agents** - Task execution, transcript analysis, strategic guidance, and framework teaching
3. **12 Utility Tools** - Development utilities automatically integrated as function-calling tools
4. **Agentic Workflows** - Multi-step task planning and execution with reasoning capabilities

### Technology Stack

- **Language**: Python 3.9+
- **AI Models**: OpenAI GPT-4o, GPT-4.1, o3-mini
- **Key Libraries**:
  - `openai>=2.7.1` - OpenAI API integration
  - `pydantic>=2.12.4` - Data validation and settings
  - `rich>=14.2.0` - Terminal UI
  - `tiktoken>=0.12.0` - Token counting
  - `pyyaml>=6.0.2` - Configuration management

### Project Purpose

This repository serves as a personal utility collection combining:
- Reusable development tools (code analysis, API testing, file operations)
- AI-powered chat interface with tool integration
- Specialized AI agents for different cognitive tasks (analysis, strategy, learning, execution)

---

## Repository Structure

```
personalUtils/
├── ChatSystem/              # Core chat system
│   ├── core/               # Core functionality
│   │   ├── chat_engine.py      # OpenAI integration, streaming, function calling
│   │   ├── conversation.py     # Memory management, context handling
│   │   └── config.py           # Pydantic settings and configuration
│   ├── tools/              # Tool integration layer
│   │   ├── tool_adapter.py     # Converts utilities to OpenAI function schemas
│   │   ├── tool_registry.py    # Tool discovery and registration
│   │   └── tool_executor.py    # Safe tool execution with error handling
│   ├── interface/          # User interfaces
│   │   └── cli.py              # Rich-powered interactive CLI
│   ├── examples/           # Example usage scripts
│   └── tests/              # ChatSystem unit tests
│
├── agents/                  # Specialized AI agents
│   ├── agent_manager.py        # Central agent orchestration
│   ├── task_executor/          # General-purpose task execution agent
│   │   ├── executor.py             # Multi-step task execution
│   │   ├── planner.py              # Task planning logic
│   │   └── reasoner.py             # Chain-of-thought reasoning
│   ├── transcript_analyzer/    # Extracts insights from transcripts
│   │   └── analyzer.py
│   ├── trillionaire_futurist/  # Strategic advisor at trillionaire scale
│   │   └── futurist.py
│   └── framework_teacher/      # Teaches through mental models/frameworks
│       └── teacher.py
│
├── tools/                   # 12 utility tools
│   ├── CodeWhisper/            # Python code analysis (LOC, complexity, structure)
│   ├── APITester/              # HTTP API endpoint testing
│   ├── DuplicateFinder/        # Find duplicate files by hash/name
│   ├── SnippetManager/         # Code snippet storage with tags
│   ├── BulkRename/             # Batch file renaming with patterns/regex
│   ├── EnvManager/             # .env file management
│   ├── FileDiff/               # File comparison and diffing
│   ├── GitStats/               # Git repository statistics
│   ├── ImportOptimizer/        # Python import optimization
│   ├── PathSketch/             # Path operations and normalization
│   ├── TodoExtractor/          # Extract TODO/FIXME comments
│   └── DataConvert/            # Convert between JSON, YAML, CSV, XML
│
├── tests/                   # Test suite
│   ├── test_agents.py          # Agent functionality tests
│   └── test_agent_switching.py # Agent switching tests
│
├── config.yaml              # Global configuration (models, tools, agents)
├── .env                     # Environment variables (API keys)
├── .env.example             # Environment template
├── requirements.txt         # Python dependencies
├── requirements-dev.txt     # Development dependencies
├── pytest.ini               # Pytest configuration
├── README.md                # Project overview
├── AGENTS_README.md         # Detailed agent documentation
├── CHATSYSTEM_SETUP.md      # Quick setup guide
└── IMPROVEMENTS.md          # Future improvements and ideas
```

### Key File Locations

| Component | Path | Purpose |
|-----------|------|---------|
| Main entry point | `ChatSystem/__main__.py` | Launch interactive chat |
| Chat engine | `ChatSystem/core/chat_engine.py` | Core OpenAI integration |
| Agent manager | `agents/agent_manager.py` | Agent selection and orchestration |
| Configuration | `config.yaml` | Models, tools, agent settings |
| Environment | `.env` | API keys and secrets |
| Tests | `pytest.ini`, `tests/` | Testing configuration and tests |

---

## Key Concepts

### 1. ChatSystem Architecture

**ChatEngine** (`ChatSystem/core/chat_engine.py`)
- Handles OpenAI API interactions
- Supports streaming responses
- Manages function calling (tool execution)
- Tracks token usage and costs

**ConversationManager** (`ChatSystem/core/conversation.py`)
- Maintains conversation history
- Manages context window limits
- Automatic context trimming when near limits
- Role-based message management (system, user, assistant, tool)

**Tool Integration** (`ChatSystem/tools/`)
- **tool_adapter.py**: Converts utility tools to OpenAI function schemas
- **tool_registry.py**: Auto-discovers tools from `tools/` directory
- **tool_executor.py**: Safely executes tool functions with error handling

### 2. Agent System

The repository includes **four specialized agents**, each with unique capabilities:

#### Agent Types (from `agents/agent_manager.py`)

| Agent | Short Name | Purpose | Use Cases |
|-------|-----------|---------|-----------|
| **Task Executor** | `executor` | General-purpose task execution | Multi-step tasks, tool operations, general problem solving |
| **Transcript Analyzer** | `analyzer` | Extract value from transcripts | Podcast analysis, interview insights, skill identification |
| **Trillionaire Futurist** | `futurist` | Strategic advisor at scale | Business decisions, opportunity analysis, high-stakes planning |
| **Framework Teacher** | `teacher` | Meta-learning specialist | Learn frameworks, develop mental models, build transferable skills |

#### Agent Configuration

Each agent has specific configuration in `config.yaml`:

```yaml
agents:
  task_executor:
    max_iterations: 5
    enable_planning: true
    enable_reasoning: true
    model: o3-mini
    timeout_seconds: 300

  transcript_analyzer:
    max_iterations: 3
    model: o3-mini
    timeout_seconds: 600

  # ... (futurist, teacher)
```

#### Agent Switching

The system supports dynamic agent switching during conversation:
- Default agent: `task_executor`
- Switch agents via CLI: `/agent analyzer`, `/agent futurist`, `/agent teacher`
- Agents share the same `ChatEngine` but maintain unique personas

### 3. Tool System

**12 Utility Tools** are automatically discovered and registered:

#### Tool Categories

**Code Analysis**
- `CodeWhisper`: Analyze Python code (LOC, functions, classes, complexity)
- `ImportOptimizer`: Optimize Python import statements
- `TodoExtractor`: Extract TODO/FIXME/HACK comments

**File Operations**
- `DuplicateFinder`: Find duplicate files by hash or filename
- `BulkRename`: Batch rename files with patterns/regex
- `FileDiff`: Compare files and show differences
- `PathSketch`: Path operations (normalize, resolve, join)

**Development Utilities**
- `APITester`: Test HTTP endpoints (GET, POST, PUT, DELETE)
- `GitStats`: Analyze git repository statistics
- `EnvManager`: Manage .env configuration files
- `SnippetManager`: Store and retrieve code snippets with tags
- `DataConvert`: Convert between JSON, YAML, CSV, XML

#### Tool Structure

Each tool follows a consistent pattern:
```
tools/ToolName/
├── README.md           # Tool documentation
├── tool_name.py        # Tool implementation
└── API_DOCS.md         # (optional) API documentation
```

### 4. Model Selection

The system supports multiple OpenAI models with different characteristics:

| Model | Config Key | Cost (per 1M tokens) | Use Case |
|-------|-----------|---------------------|----------|
| `gpt-4o-mini` | `simple` | $0.15 / $0.60 | Fast, cost-effective tasks |
| `gpt-4o` | `general` | $2.50 / $10.00 | Best balance (default) |
| `gpt-4.1` | `long_context` | $2.00 / $8.00 | Large codebases (1M token context) |
| `o3-mini` | `reasoning` | $1.00 / $4.00 | Advanced reasoning tasks |

---

## Development Workflows

### Working with ChatSystem

#### 1. Interactive CLI

```bash
# Launch interactive chat
python -m ChatSystem

# Available commands
/help          # Show all commands
/tools         # List available tools
/agents        # List available agents
/agent <name>  # Switch to specific agent
/stats         # View usage statistics
/context       # Show context window usage
/clear         # Clear conversation history
/export        # Export conversation
/exit          # Exit chat
```

#### 2. Programmatic Usage

```python
from ChatSystem import ChatEngine, get_settings
from agents.agent_manager import AgentManager, AgentType

# Initialize
settings = get_settings()
engine = ChatEngine(settings=settings)

# Basic chat
for chunk in engine.chat("Hello!"):
    print(chunk, end="")

# Using agents
manager = AgentManager(settings=settings)
agent = manager.get_agent(AgentType.TASK_EXECUTOR, chat_engine=engine)

# Execute complex task
result = agent.execute_task("Analyze all Python files and extract TODOs")
```

#### 3. Adding New Tools

To add a new tool:

1. Create directory: `tools/NewTool/`
2. Implement tool: `tools/NewTool/new_tool.py`
3. Add to config: Update `config.yaml` tools.enabled list
4. Tool will be auto-discovered by `ToolRegistry`

Tool implementation pattern:
```python
class NewTool:
    """Tool description for OpenAI function calling."""

    def execute(self, param1: str, param2: int) -> dict:
        """
        Execute the tool.

        Args:
            param1: Description of param1
            param2: Description of param2

        Returns:
            Dictionary with results
        """
        # Implementation
        return {"status": "success", "result": "..."}
```

### Working with Agents

#### Agent Development Pattern

Each agent follows this structure:

```python
class CustomAgent:
    """Agent description."""

    # System persona that defines agent behavior
    SYSTEM_PERSONA = """
    You are [agent identity and role].

    Your capabilities:
    - [capability 1]
    - [capability 2]

    Your approach:
    - [approach guideline 1]
    - [approach guideline 2]
    """

    def __init__(self, chat_engine, settings, max_iterations=5):
        self.chat_engine = chat_engine
        self.settings = settings
        self.max_iterations = max_iterations

        # Add system persona to conversation
        self.chat_engine.conversation.add_message("system", self.SYSTEM_PERSONA)

    def execute(self, task: str) -> str:
        """Execute agent-specific task."""
        # Implementation
        pass
```

#### Agent Selection Guidelines

**When to use each agent:**

- **Task Executor**: Multi-step tasks, tool operations, general problem-solving
- **Transcript Analyzer**: Extracting insights from podcasts, interviews, talks
- **Trillionaire Futurist**: Strategic decisions, opportunity evaluation, high-stakes planning
- **Framework Teacher**: Learning frameworks, developing mental models, meta-skills

---

## Code Conventions

### Python Style

- **Python Version**: 3.9+ required
- **Type Hints**: Use type hints for all function parameters and returns
- **Docstrings**: Google-style docstrings for all public functions/classes
- **Formatting**: Follow PEP 8 guidelines
- **Imports**: Group imports (standard library, third-party, local)

### Configuration Management

**Environment Variables** (`.env`)
```env
OPENAI_API_KEY=sk-your-key-here
MODEL_NAME=gpt-4o
```

**YAML Configuration** (`config.yaml`)
```yaml
models:
  simple: gpt-4o-mini
  general: gpt-4o

tools:
  enabled:
    - CodeWhisper
    - APITester

agent:
  max_iterations: 5
  enable_planning: true
```

**Pydantic Settings** (`ChatSystem/core/config.py`)
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    model_name: str = "gpt-4o"

    class Config:
        env_file = ".env"
```

### Error Handling

**Tool Execution**
- All tool executions wrapped in try-except
- Return structured error responses
- Log errors for debugging

```python
try:
    result = tool.execute(**params)
    return {"status": "success", "result": result}
except Exception as e:
    return {"status": "error", "error": str(e)}
```

**Agent Execution**
- Max iterations prevent infinite loops
- Timeout handling for long-running tasks
- Graceful degradation on failures

### File Organization

- **Single Responsibility**: Each module has one clear purpose
- **Separation of Concerns**: Core, tools, agents, interface layers are separate
- **Discoverability**: Tools auto-discovered from `tools/` directory
- **Documentation**: Each tool/agent includes README.md

---

## Testing

### Test Framework

**pytest** configuration in `pytest.ini`:

```ini
[pytest]
python_files = test_*.py
python_classes = Test*
python_functions = test_*
testpaths = tests

addopts = -v --tb=short --color=yes

markers =
    slow: slow tests
    integration: integration tests
    unit: unit tests
```

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_agents.py

# Run with markers
pytest -m unit          # Only unit tests
pytest -m "not slow"    # Skip slow tests

# Run with coverage
pytest --cov=. --cov-report=html
```

### Test Structure

```python
import pytest
from agents.agent_manager import AgentManager, AgentType

def test_agent_creation():
    """Test agent creation and initialization."""
    manager = AgentManager()
    agent = manager.get_agent(AgentType.TASK_EXECUTOR)
    assert agent is not None

def test_agent_switching():
    """Test dynamic agent switching."""
    manager = AgentManager()
    manager.set_current_agent(AgentType.TRANSCRIPT_ANALYZER)
    assert manager.current_agent_type == AgentType.TRANSCRIPT_ANALYZER
```

### Test Files

- `tests/test_agents.py` - Agent functionality tests
- `tests/test_agent_switching.py` - Agent switching tests
- `ChatSystem/tests/` - ChatSystem unit tests

---

## Configuration

### Environment Setup

**Initial Setup**
```bash
# 1. Clone repository
git clone <repo-url>
cd personalUtils

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
nano .env  # Add OPENAI_API_KEY

# 4. Test installation
python -c "from ChatSystem import get_settings; print('✓ Success!')"

# 5. Run interactive chat
python -m ChatSystem
```

### Configuration Files

#### `.env` - Environment Variables
```env
# Required
OPENAI_API_KEY=sk-your-actual-key-here

# Optional (defaults shown)
MODEL_NAME=gpt-4o
```

#### `config.yaml` - System Configuration

```yaml
# Model selection
models:
  simple: gpt-4o-mini        # Fast, cheap
  general: gpt-4o            # Balanced (default)
  long_context: gpt-4.1      # Large context (1M tokens)
  reasoning: o3-mini         # Advanced reasoning

# Tool configuration
tools:
  enabled:
    - CodeWhisper
    - APITester
    - DuplicateFinder
    - SnippetManager
    - BulkRename
    - EnvManager
    - FileDiff
    - GitStats
    - ImportOptimizer
    - PathSketch
    - TodoExtractor
    - DataConvert

# Default agent configuration
agent:
  max_iterations: 5
  enable_planning: true
  enable_reasoning: true
  timeout_seconds: 300
  default_agent: task_executor

# Agent-specific configurations
agents:
  task_executor:
    max_iterations: 5
    enable_planning: true
    enable_reasoning: true
    model: o3-mini
    timeout_seconds: 300

  transcript_analyzer:
    max_iterations: 3
    enable_planning: false
    enable_reasoning: true
    model: o3-mini
    timeout_seconds: 600

  trillionaire_futurist:
    max_iterations: 5
    enable_planning: true
    enable_reasoning: true
    model: o3-mini
    timeout_seconds: 600

  framework_teacher:
    max_iterations: 3
    enable_planning: false
    enable_reasoning: true
    model: o3-mini
    timeout_seconds: 300

# CLI configuration
cli:
  theme: monokai
  show_timestamps: true
  show_token_usage: true
  auto_save_history: true
```

### Cost Optimization

**Model Selection Strategy**
- Use `gpt-4o-mini` for simple queries (70% cheaper)
- Use `gpt-4o` for balanced performance (default)
- Use `o3-mini` for complex reasoning (agents use this)
- Use `gpt-4.1` only for large codebases

**Token Management**
- Monitor usage: `/stats` command in CLI
- Context auto-trims at 95% capacity
- Manual trim: `conversation.trim_context(target_tokens=50000)`

---

## Common Tasks

### For AI Assistants Working on This Codebase

#### Task 1: Analyze Code Structure
```bash
# Use CodeWhisper tool via CLI
python -m ChatSystem
> Analyze the code structure in ChatSystem/core
```

#### Task 2: Add a New Utility Tool
1. Create tool directory: `tools/NewTool/`
2. Implement tool: `tools/NewTool/new_tool.py`
3. Add README: `tools/NewTool/README.md`
4. Update config: Add to `config.yaml` tools.enabled
5. Test tool: Create test in `tests/`

#### Task 3: Modify Agent Behavior
1. Locate agent: `agents/<agent_name>/`
2. Modify `SYSTEM_PERSONA` to change behavior
3. Update agent logic in `execute()` or `respond()` method
4. Test changes: `python test_agents.py`

#### Task 4: Add New Model Support
1. Update `config.yaml` models section
2. Update `ChatSystem/core/config.py` if needed
3. Update documentation in README

#### Task 5: Debug Tool Integration
1. Check tool registry: `python -m ChatSystem`, then `/tools`
2. Verify tool adapter: `ChatSystem/tools/tool_adapter.py`
3. Check execution logs in `ChatSystem/tools/tool_executor.py`
4. Test tool manually:
```python
from tools.CodeWhisper.code_whisper import CodeWhisper
tool = CodeWhisper()
result = tool.execute(directory="./ChatSystem")
print(result)
```

#### Task 6: Optimize Token Usage
1. Check current usage: `/stats` in CLI
2. Review conversation length: `/context`
3. Adjust max_iterations in config for agents
4. Implement context trimming: `conversation.trim_context()`

#### Task 7: Add New Agent
1. Create agent directory: `agents/new_agent/`
2. Implement agent class with `SYSTEM_PERSONA`
3. Add to `AgentType` enum in `agent_manager.py`
4. Add to `AGENT_DESCRIPTIONS` in `agent_manager.py`
5. Update config: Add agent config to `config.yaml`
6. Update documentation: `AGENTS_README.md`

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "OpenAI API key not configured"
**Cause**: Missing or invalid API key in `.env`
**Solution**:
```bash
# Check .env file exists
ls -la .env

# Add API key
echo "OPENAI_API_KEY=sk-your-key-here" > .env
```

#### Issue: "Module not found" errors
**Cause**: Dependencies not installed
**Solution**:
```bash
pip install -r requirements.txt
```

#### Issue: "Context window at 95%"
**Cause**: Conversation too long, approaching token limit
**Solution**:
```python
# Use /clear command in CLI, or programmatically:
conversation.trim_context(target_tokens=50000)
```

#### Issue: Rate limit errors
**Cause**: Too many API requests
**Solution**:
- Switch to `gpt-4o-mini` for lower rate limits
- Add delays between requests
- Reduce `max_iterations` in agent config

#### Issue: Tool execution failures
**Cause**: Tool errors or invalid parameters
**Solution**:
1. Check tool documentation: `tools/<ToolName>/README.md`
2. Test tool manually:
```python
from tools.ToolName.tool_name import ToolName
tool = ToolName()
result = tool.execute(param1="value")
```
3. Check tool executor logs: `ChatSystem/tools/tool_executor.py`

#### Issue: Agent not switching properly
**Cause**: Agent manager state issues
**Solution**:
```python
# Reset agent manager
manager = AgentManager()
manager.set_current_agent(AgentType.DESIRED_AGENT, chat_engine=engine)
```

#### Issue: Tests failing
**Cause**: Environment or dependency issues
**Solution**:
```bash
# Reinstall dependencies
pip install -r requirements.txt -r requirements-dev.txt

# Run tests with verbose output
pytest -v -s

# Run specific test
pytest tests/test_agents.py::test_specific_function -v
```

### Debug Mode

Enable verbose logging for debugging:

```python
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# Run code
engine = ChatEngine(settings=settings)
```

---

## Best Practices for AI Assistants

### When Making Changes

1. **Understand Before Modifying**
   - Read relevant documentation first
   - Review related code sections
   - Check existing tests

2. **Maintain Consistency**
   - Follow existing code patterns
   - Use same naming conventions
   - Match docstring style

3. **Test Changes**
   - Run existing tests: `pytest`
   - Add new tests for new features
   - Verify manually with CLI

4. **Update Documentation**
   - Update README.md if user-facing changes
   - Update AGENTS_README.md for agent changes
   - Update CLAUDE.md (this file) for structural changes

### Code Review Checklist

Before suggesting changes:
- [ ] Type hints present
- [ ] Docstrings complete
- [ ] Error handling implemented
- [ ] Tests added/updated
- [ ] Configuration updated if needed
- [ ] Documentation updated
- [ ] No hardcoded secrets
- [ ] Follows existing patterns

### Communication Guidelines

When explaining code:
- Reference specific file paths: `ChatSystem/core/chat_engine.py:142`
- Provide context about why code exists
- Explain trade-offs in design decisions
- Suggest alternatives when appropriate

---

## Additional Resources

### Documentation Files

- **README.md** - Project overview and quick start
- **AGENTS_README.md** - Detailed agent documentation with examples
- **CHATSYSTEM_SETUP.md** - Quick setup guide for ChatSystem
- **IMPROVEMENTS.md** - Future improvements and feature ideas
- **PR_REVIEW_FINAL.md** - Pull request review documentation

### Tool Documentation

Each tool has its own README:
- `tools/CodeWhisper/README.md`
- `tools/APITester/README.md`
- ... (12 tools total)

### Example Scripts

- `ChatSystem/examples/basic_chat.py` - Basic chat usage
- `ChatSystem/examples/tool_usage.py` - Tool integration examples
- `ChatSystem/examples/agentic_workflow.py` - Multi-step agent workflow

### Git History

Recent significant commits:
- `f285062` - Merge three agent system PR
- `da3a824` - Add agent switching test case
- `7eaac47` - Add three specialized AI agents with multi-agent orchestration
- `f234a4b` - Reorganize repository structure for better modularity

---

## Quick Reference

### Essential Commands

```bash
# Run interactive chat
python -m ChatSystem

# Run tests
pytest

# Test specific agent
python test_agents.py

# Install dependencies
pip install -r requirements.txt
```

### Key Python Imports

```python
# ChatSystem
from ChatSystem import ChatEngine, get_settings
from ChatSystem.core import ConversationManager
from ChatSystem.tools import ToolRegistry

# Agents
from agents.agent_manager import AgentManager, AgentType

# Tools
from tools.CodeWhisper.code_whisper import CodeWhisper
# ... (other tools)
```

### Configuration Paths

```
.env                        # API keys
config.yaml                 # System configuration
pytest.ini                  # Test configuration
requirements.txt            # Dependencies
```

---

## Version Information

- **Python**: 3.9+
- **OpenAI API**: 2.7.1+
- **Pydantic**: 2.12.4+
- **Rich**: 14.2.0+
- **Pytest**: Latest (for testing)

---

**Last Updated**: 2025-11-14
**Repository**: personalUtils
**Branch**: claude/claude-md-mhz9e1q9iage91um-01GgukA1S4ZkisQ8cNaVEydE

For questions or issues, refer to the documentation files or examine the test suite for usage examples.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vkstark) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
