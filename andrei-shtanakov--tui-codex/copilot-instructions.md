## tui-codex

> A Terminal User Interface (TUI) application for dynamically creating, managing, and interacting with AutoGen AI agents. The application provides a visual interface for configuring agents with custom parameters, assigning tasks, and viewing results in real-time.

# AutoGen Agent Manager - TUI Application

## Project Overview

A Terminal User Interface (TUI) application for dynamically creating, managing, and interacting with AutoGen AI agents. The application provides a visual interface for configuring agents with custom parameters, assigning tasks, and viewing results in real-time.

## Technology Stack

### Backend & Core
- **Python 3.12+**
- **AutoGen** (`autogen-agentchat`, `autogen-ext`) - AI agent framework
- **Textual** - Terminal User Interface framework
- **Rich** - Terminal formatting and styling
- **SQLite** - Local database for persistence
- **SQLAlchemy** - ORM for database operations

### AI & Models
- **OpenAI GPT-4o / GPT-4o-mini** - Language models
- **Anthropic Claude** (optional) - Alternative model provider

## Core Development Rules

### 1. Package Management
- **ONLY use `uv`**, NEVER pip
- Installation: `uv add package`
- Running tools: `uv run tool`
- Dev dependencies: `uv add --dev package`
- Upgrading: `uv add --upgrade-package package`
- **FORBIDDEN**: `uv pip install`, `@latest` syntax

### 2. Code Quality Standards
- Type hints required for ALL functions and methods
- Use `pyrefly` for type checking:
  - `pyrefly init` - Initialize type checking
  - `pyrefly check` - Check after every change
- Public APIs must have comprehensive docstrings
- Functions must be focused and small (single responsibility)
- Maximum line length: 88 characters
- Use `ruff` for linting: `uv run ruff check .`

### 3. Testing Requirements
- Framework: `uv run pytest`
- Async testing: use `anyio`, NOT `asyncio` directly
- All new features require corresponding tests
- Bug fixes require regression tests
- Minimum test coverage: 80%

### 4. Code Style
- PEP 8 naming conventions:
  - `snake_case` for functions and variables
  - `PascalCase` for class names
  - `UPPER_SNAKE_CASE` for constants
- Use f-strings for string formatting
- Document all modules, classes, and public functions with docstrings
- Follow Google docstring format

## Application Architecture

### UI Layout (4-Column Design)

```
┌─────────────┬──────────────────┬──────────────────┬─────────────┐
│   Agents    │  Agent Config    │  Task & Results  │   Tools     │
│   List      │  Parameters      │                  │   Library   │
├─────────────┼──────────────────┼──────────────────┼─────────────┤
│             │                  │                  │             │
│ • Agent 1   │ Name: _______    │ Task Input:      │ ☐ get_weather│
│ • Agent 2   │                  │ ____________     │ ☐ calculate │
│ • Agent 3   │ System Message:  │ ____________     │ ☐ web_search│
│             │ ____________     │                  │ ☐ file_ops  │
│ [+ Add]     │ ____________     │ [Run Task]       │ ☐ database  │
│ [✎ Edit]    │                  │                  │             │
│ [× Delete]  │ Model:           │ ──────────────── │ [+ Add Tool]│
│             │ • gpt-4o ▼       │ Results:         │             │
│             │                  │                  │             │
│             │ ☑ Streaming      │ Loading...       │             │
│             │ ☐ Tools          │ or               │             │
│             │                  │ Agent response   │             │
│             │ Temperature:     │ displayed here   │             │
│             │ ━━━●━━━━━ 0.7    │                  │             │
│             │                  │                  │             │
│             │ [Save] [Cancel]  │ [Clear]          │             │
└─────────────┴──────────────────┴──────────────────┴─────────────┘
```

### Column Specifications

#### Column 1: Agents List (Left - 20% width)
- Display list of all created agents
- Show agent name and brief status (active/idle)
- Buttons:
  - `[+ Add]` - Create new agent
  - `[✎ Edit]` - Modify selected agent
  - `[× Delete]` - Remove agent with confirmation
- Highlight selected agent
- Show agent count at bottom

#### Column 2: Agent Configuration (Center-Left - 30% width)
- **Name Field** - Text input for agent identifier
- **System Message** - Multi-line text area for agent instructions
- **Model Selection** - Dropdown menu:
  - gpt-4o
  - gpt-4o-mini
  - claude-sonnet-4-5
  - claude-opus-4
- **Streaming Toggle** - Checkbox for token streaming
- **Enable Tools** - Checkbox to allow tool usage
- **Temperature Slider** - Range 0.0 to 2.0 (default 1.0)
- **Max Tokens** - Optional numeric input
- **Save/Cancel Buttons**

#### Column 3: Task & Results (Center-Right - 35% width)
- **Task Input Area** - Multi-line text area for task description
- **[Run Task] Button** - Execute agent with given task
- **Results Display Area**:
  - Show streaming output if enabled
  - Display complete response
  - Show metadata (tokens used, execution time)
  - Support markdown rendering
- **[Clear] Button** - Clear results area
- **[Export] Button** - Save conversation to file

#### Column 4: Tools Library (Right - 15% width)
- Display available tools with checkboxes
- Built-in tools:
  - `get_weather` - Weather information
  - `calculate` - Math operations
  - `web_search` - Internet search
  - `file_read` - Read files
  - `file_write` - Write files
  - `execute_code` - Run Python code
- **[+ Add Tool]** - Define custom tools
- **[✎ Edit Tool]** - Modify tool implementation
- Show tool description on hover/selection

## Core Features

### 1. Agent Management
- **Create** agents with custom configurations
- **Edit** existing agent parameters
- **Delete** agents with confirmation dialog
- **List** all agents with status indicators
- **Persist** agent configurations to database
- **Duplicate** existing agents
- **Import/Export** agent configurations as JSON

### 2. Agent Configuration
- Dynamic parameter setting:
  - Name (unique identifier)
  - System message (role and instructions)
  - Model selection (multiple providers)
  - Streaming mode (on/off)
  - Tool usage (enable/disable)
  - Temperature control
  - Max tokens limit
- Validation of all inputs
- Preview mode before saving

### 3. Task Execution
- Run tasks against selected agents
- Support for streaming responses
- Display real-time output
- Show execution metadata
- Handle errors gracefully
- Cancel running tasks
- Task history per agent

### 4. Tool Management
- Define custom tools as Python functions
- Enable/disable tools per agent
- Tool library management
- Tool parameter validation
- Tool execution tracking
- Import tools from files

### 5. Data Persistence
- Store agent configurations in SQLite
- Save conversation history
- Export/import capabilities
- Configuration versioning

## Database Schema

### Table: agents
- `id` - INTEGER PRIMARY KEY
- `name` - TEXT UNIQUE NOT NULL
- `system_message` - TEXT NOT NULL
- `model` - TEXT NOT NULL
- `streaming` - BOOLEAN DEFAULT FALSE
- `tools_enabled` - BOOLEAN DEFAULT FALSE
- `temperature` - REAL DEFAULT 1.0
- `max_tokens` - INTEGER NULL
- `created_at` - TIMESTAMP
- `updated_at` - TIMESTAMP

### Table: tools
- `id` - INTEGER PRIMARY KEY
- `name` - TEXT UNIQUE NOT NULL
- `description` - TEXT
- `implementation` - TEXT NOT NULL
- `enabled` - BOOLEAN DEFAULT TRUE
- `created_at` - TIMESTAMP

### Table: agent_tools (Many-to-Many)
- `agent_id` - INTEGER FOREIGN KEY
- `tool_id` - INTEGER FOREIGN KEY
- PRIMARY KEY (agent_id, tool_id)

### Table: task_history
- `id` - INTEGER PRIMARY KEY
- `agent_id` - INTEGER FOREIGN KEY
- `task` - TEXT NOT NULL
- `result` - TEXT
- `tokens_used` - INTEGER
- `execution_time` - REAL
- `status` - TEXT (success/error/cancelled)
- `created_at` - TIMESTAMP

## Project Structure

```
autogen-agent-manager/
├── pyproject.toml              # Project dependencies
├── CLAUDE.md                   # This file
├── README.md                   # User documentation
├── .python-version             # Python version specification
├── src/
│   ├── __init__.py
│   ├── main.py                 # Application entry point
│   ├── app.py                  # Main Textual app class
│   ├── database/
│   │   ├── __init__.py
│   │   ├── models.py           # SQLAlchemy models
│   │   └── database.py         # Database connection
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── factory.py          # Agent creation factory
│   │   ├── manager.py          # Agent lifecycle management
│   │   └── executor.py         # Task execution logic
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── registry.py         # Tool registration
│   │   ├── builtin.py          # Built-in tools
│   │   └── custom.py           # Custom tool loader
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── screens/
│   │   │   ├── __init__.py
│   │   │   └── main_screen.py  # Main application screen
│   │   ├── widgets/
│   │   │   ├── __init__.py
│   │   │   ├── agent_list.py   # Column 1
│   │   │   ├── agent_config.py # Column 2
│   │   │   ├── task_panel.py   # Column 3
│   │   │   └── tools_panel.py  # Column 4
│   │   └── styles.py           # CSS styling
│   └── utils/
│       ├── __init__.py
│       ├── config.py           # Configuration management
│       └── validators.py       # Input validation
├── tests/
│   ├── __init__.py
│   ├── test_agents.py
│   ├── test_tools.py
│   ├── test_database.py
│   └── test_ui.py
└── data/
    └── agents.db               # SQLite database
```

## Common Commands

### Project Setup
```bash
# Clone repository
git clone <repository-url>
cd autogen-agent-manager

# Install dependencies
uv sync

# Initialize database
uv run python -m src.database.init_db
```

### Development
```bash
# Run application
uv run python -m src.main

# Run with development mode
uv run textual run --dev src.main:app

# Type checking
pyrefly check

# Linting
uv run ruff check .

# Auto-fix linting issues
uv run ruff check --fix .

# Format code
uv run ruff format .
```

### Testing
```bash
# Run all tests
uv run pytest

# Run with coverage
uv run pytest --cov=src --cov-report=html

# Run specific test file
uv run pytest tests/test_agents.py

# Run with verbose output
uv run pytest -v

# Run tests in watch mode
uv run pytest-watch
```

### Database Management
```bash
# Create migration
uv run alembic revision --autogenerate -m "description"

# Apply migrations
uv run alembic upgrade head

# Rollback migration
uv run alembic downgrade -1

# Reset database
uv run python -m src.database.reset_db
```

### Dependencies
```bash
# Add new dependency
uv add package-name

# Add development dependency
uv add --dev package-name

# Update dependency
uv add --upgrade-package package-name

# Remove dependency
uv remove package-name

# Show dependency tree
uv tree
```

## Key Dependencies

### Core
```toml
[project]
dependencies = [
    "textual>=0.80.0",
    "rich>=13.7.0",
    "autogen-agentchat>=0.4.0",
    "autogen-ext[openai]>=0.4.0",
    "sqlalchemy>=2.0.0",
    "alembic>=1.13.0",
]

[tool.uv]
dev-dependencies = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=4.1.0",
    "anyio>=4.0.0",
    "ruff>=0.5.0",
]
```

## Configuration Management

### Environment Variables
```bash
# Required
OPENAI_API_KEY=sk-...

# Optional
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=sqlite:///data/agents.db
LOG_LEVEL=INFO
MAX_CONCURRENT_AGENTS=5
```

### Configuration File
Location: `config.yaml` or `~/.config/autogen-manager/config.yaml`

```yaml
# Configuration structure (not actual YAML)
database:
  url: sqlite:///data/agents.db
  
models:
  default: gpt-4o
  available:
    - gpt-4o
    - gpt-4o-mini
    - claude-sonnet-4-5

ui:
  theme: dark
  refresh_rate: 60
  
agents:
  max_concurrent: 5
  default_temperature: 1.0
  default_streaming: false
```

## Feature Priorities

### Phase 1 (MVP)
- [x] Basic UI layout with 4 columns
- [ ] Agent CRUD operations
- [ ] Simple agent configuration (name, system_message, model)
- [ ] Task execution without tools
- [ ] SQLite persistence
- [ ] Basic error handling

### Phase 2
- [ ] Tool management system
- [ ] Enable/disable tools per agent
- [ ] Built-in tools library
- [ ] Streaming output support
- [ ] Task history
- [ ] Export/import configurations

### Phase 3
- [ ] Custom tool creation
- [ ] Advanced agent parameters (temperature, max_tokens)
- [ ] Multi-agent conversations
- [ ] Conversation branching
- [ ] Performance metrics
- [ ] Search and filter agents

### Phase 4
- [ ] Plugin system for tools
- [ ] Remote model providers
- [ ] Collaborative features
- [ ] Advanced visualization
- [ ] Agent templates library
- [ ] API server mode

## Error Handling

### Categories
1. **Input Validation Errors** - Invalid user input
2. **Model Errors** - API failures, rate limits
3. **Database Errors** - Connection issues, constraint violations
4. **Tool Execution Errors** - Runtime failures in tools
5. **UI Errors** - Rendering or interaction issues

### Strategy
- Use try-except blocks with specific exception types
- Log all errors with context
- Show user-friendly error messages in UI
- Provide recovery options when possible
- Never expose sensitive information in errors

## Performance Considerations

- Lazy load agent configurations
- Cache tool definitions
- Use async/await for all I/O operations
- Implement connection pooling for database
- Stream large responses to UI
- Limit concurrent agent executions
- Implement request debouncing for UI inputs

## Security Guidelines

- Never store API keys in database
- Use environment variables for secrets
- Sanitize all user inputs
- Validate tool code before execution
- Implement rate limiting for task execution
- Log security-relevant events
- Regular dependency updates

## Logging

### Log Levels
- **DEBUG** - Detailed diagnostic information
- **INFO** - General informational messages
- **WARNING** - Warning messages for recoverable issues
- **ERROR** - Error messages for failures
- **CRITICAL** - Critical failures requiring immediate attention

### Log Format
```
[TIMESTAMP] [LEVEL] [MODULE] Message
```

### Log Locations
- Console output (stdout/stderr)
- File: `logs/autogen-manager.log`
- Rotating logs (max 10MB, keep 5 files)

## Documentation Requirements

- Docstrings for all public functions and classes
- README.md with installation and usage instructions
- CHANGELOG.md for version history
- API documentation for tool development
- User guide with screenshots
- Developer guide for contributors

## Textual-Specific Guidelines

### CSS Styling
- Define styles in separate `styles.py` or inline
- Use Textual's CSS syntax
- Follow consistent color scheme
- Ensure accessibility (contrast ratios)
- Support both dark and light themes

### Widget Development
- Extend appropriate base widgets
- Implement reactive properties
- Handle keyboard shortcuts
- Support mouse interactions
- Emit custom messages for parent communication

### Layout Management
- Use Grid or Horizontal/Vertical containers
- Make responsive to terminal size
- Handle minimum size requirements
- Test on different terminal sizes

## Contributing Guidelines

1. Fork the repository
2. Create a feature branch
3. Follow code style guidelines
4. Write tests for new features
5. Run `pyrefly check` before committing
6. Submit pull request with description
7. Address review feedback

## License

MIT License - See LICENSE file for details

## Maintenance

### Regular Tasks
- Update dependencies monthly
- Review and close stale issues
- Update documentation for new features
- Run security audits
- Performance profiling quarterly

### Versioning
Follow Semantic Versioning (SemVer):
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

## Support

- GitHub Issues for bugs and feature requests
- Discussions for questions and ideas
- Documentation wiki for guides
- Email support for critical issues

---

**Last Updated**: 2025-11-01  
**Version**: 1.0.0  
**Maintainer**: Project Team

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrei-shtanakov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andrei-shtanakov)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
