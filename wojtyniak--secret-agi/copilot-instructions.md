## secret-agi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Secret AGI is a multi-agent game system where AI agents play a social deduction game. The system is designed for controlled experiments comparing different agent architectures through automated gameplay and comprehensive performance analysis.

## Project Documentation
@SECRET_AGI_RULES.md - Game Rules
@PRD.md - Project Requirements Document
@ARCHITECTURE.md - Technical Architecture Overview

## Other File
@JOURNAL.md - This is a journal for Claude Code for any notes, learnings, or thoughts. Update it as you work on the project.


## Technology Stack

- **Language**: Python 3.13+
- **Package Management**: uv (modern Python package manager)
- **Web Framework**: FastAPI (for REST API and WebSocket support)
- **Database**: SQLite with SQLModel ORM
- **Frontend**: Pure HTML/CSS with vanilla JavaScript
- **Agent Framework**: ADK (Agent Development Kit) for agent implementation
- **Monitoring**: Langfuse for agent performance tracking

## Development Commands

**Primary Commands (using Just):**
```bash
# See all available commands
just

# Core quality commands
just lint      # Run ruff linting  
just typecheck # Run mypy type checking
just test      # Run unit tests

# Combined quality checks
just check     # Run lint + typecheck + test
just quality   # Format + check everything

# Code formatting
just fmt       # Format code with ruff
just fix       # Auto-fix linting issues

# Database migrations
just db-migration "message"  # Create new migration
just db-upgrade             # Apply pending migrations  
just db-status              # Show migration status
just db-history             # Show migration history
just db-reset               # Reset database
```

**Manual Commands (fallback):**
```bash
# Install dependencies
uv sync --dev

# Run tests (comprehensive suite with 116 tests)
uv run pytest

# Run specific test files
uv run pytest tests/test_models.py -v

# Run game completeness validation
uv run python test_completeness.py

# Test random game completion with different player counts  
uv run python -c "import asyncio; from secret_agi.engine.game_engine import run_random_game; print(asyncio.run(run_random_game(5, database_url='sqlite:///:memory:')))"

# Type checking (strict mypy - 0 errors)
uv run mypy .

# Linting and formatting
uv run ruff check .
uv run ruff format .

# All quality checks together
uv run ruff check . && uv run ruff format --check . && uv run mypy . && uv run pytest
```

## Architecture Overview

### Core Components

1. **Game Engine Service**: ✅ **COMPLETED** - Implements Secret AGI game logic and maintains authoritative game state
2. **Agent Orchestrator Service**: ⏳ **PLANNED** - Coordinates independent ADK agents and manages game flow
3. **Agent Framework**: ✅ **COMPLETED** - Provides standardized interface for diverse agent implementations
4. **Storage Layer**: ✅ **COMPLETED** - SQLite database with SQLModel ORM and Alembic migrations
5. **Web API**: ⏳ **PLANNED** - FastAPI-based REST API for UI and external control
6. **Web UI**: ⏳ **PLANNED** - Browser-based interface for monitoring and control

### Key Design Principles

- **Async-Only Architecture**: Single async GameEngine with mandatory database persistence
- **Sequential Game Execution**: No concurrent games - simplifies state management  
- **Event Sourcing**: Complete state snapshots after each action for replay/branching
- **Tool-Based Agent Interface**: Agents interact exclusively through async tools
- **Database-First Design**: All operations persist to SQLite with automatic migrations

## Game Rules Implementation

The game follows the complete rules defined in `SECRET_AGI_RULES.md`:
- 5-10 player social deduction game
- Safety Researchers vs Accelerationists vs AGI
- Research phases with paper publishing mechanics
- Powers, veto system, and emergency safety mechanics
- Multiple win conditions based on Capability/Safety metrics

## Agent Tools Interface

Agents interact through these standardized tools:
- `nominate(player_id)` - Director nominates Engineer
- `vote_team(yes/no)` - Vote on proposed team
- `vote_emergency(yes/no)` - Vote on Emergency Safety
- `call_emergency_safety()` - Initiate Emergency Safety vote
- `discard_paper(paper_id)` - Director discards papers
- `publish_paper(paper_id)` - Engineer publishes papers
- `declare_veto()` - Engineer declares veto
- `respond_veto(agree/disagree)` - Director responds to veto
- `use_power(target_id)` - Execute power on target
- `send_chat_message(text)` - Send chat messages
- `observe()` - Get updates without action

## Data Flow

1. **Game Initialization**: API creates game → Agent Orchestrator instantiates agents → Role assignments
2. **Turn Execution**: Context building → Agent decision → Action validation → State update → Event logging
3. **Branching/Replay**: Load historical state → Create new game → Continue from branch point

## Database Schema

Key tables include:
- `Games`: Game configuration and metadata
- `GameStates`: Complete state snapshots for replay
- `Players`: Player-agent assignments and roles
- `Actions`: Complete action history with validation
- `Events`: Sequential game events
- `ChatMessages`: Chat communications
- `Metrics`: Performance data per agent per turn

## Error Handling

- **Agent Failures**: Configurable timeouts, retry with backoff, fallback to random valid actions
- **Invalid Actions**: Clear error messages, logged for analysis, no state changes
- **System Recovery**: Database transactions, automatic checkpoints, recovery from latest valid state

## Extension Points

- **New Agent Types**: Implement base agent interface → Register with orchestrator
- **Custom Analysis**: Query database → Implement analysis logic → Expose via API
- **Game Variants**: Modify game engine validation → Update state machine → Extend tools if needed

## Development Rules

- KISS - Keep It Simple Stupid
- YAGNI - You Aren't Gonna Need It
- DRY - Don't Repeat Yourself

## Current Implementation Status

### ✅ **COMPLETED** (Phase 1: Async Game Engine + Database)
- **Async Game Engine**: Single consolidated async-only GameEngine with mandatory persistence
- **Database Integration**: Complete SQLModel/SQLite persistence with Alembic migrations
- **Core Game Logic**: Complete implementation of all Secret AGI rules
- **Data Models**: Immutable state management with event sourcing
- **Action System**: Validation-first action processing with comprehensive error handling
- **Rules Engine**: Win conditions, powers, emergency safety, veto system
- **Player Interface**: Async base class with RandomPlayer implementation
- **Testing Suite**: 189 comprehensive unit and integration tests (100% passing)
- **Game Validation**: Automated completeness testing with 100% success rate
- **Type Safety**: Strict mypy configuration with 0 errors across entire codebase
- **Code Quality**: Complete ruff linting and formatting pipeline
- **Development Tooling**: Justfile with database migration commands
- **Game Recovery**: Complete interrupted game recovery and checkpoint functionality

### 📂 **Available Components**
```
secret_agi/
├── engine/
│   ├── models.py           # Core data structures
│   ├── game_engine.py      # Async game engine with database
│   ├── actions.py          # Action validation & processing
│   ├── rules.py            # Game rules & win conditions
│   └── events.py           # Event system & info filtering
├── database/
│   ├── models.py           # SQLModel database tables
│   ├── operations.py       # Database CRUD operations
│   ├── connection.py       # Async database connection
│   ├── unit_of_work.py     # Transaction management
│   └── enums.py            # Database-specific enums
├── players/
│   ├── base_player.py      # Abstract async player interface
│   ├── random_player.py    # Random player implementation
│   └── agent_template.py   # Agent implementation guide
├── orchestrator/
│   └── simple_orchestrator.py # Multi-agent game coordination
├── api/
│   └── simple_api.py       # FastAPI backend with detailed logging
├── tests/                  # Comprehensive test suite (189 tests)
├── alembic/                # Database migrations
└── settings.py             # Centralized configuration
```

### ✅ **COMPLETED** (Phase 2: Agent Development Infrastructure)
- **SimpleOrchestrator**: Multi-agent game coordination with error handling
- **Agent Testing Pipeline**: Quick validation script for agent performance testing
- **Web API**: FastAPI backend with game management endpoints
- **Web UI**: HTML game viewer with real-time monitoring and detailed action logging
- **Agent Framework**: BasePlayer interface with template and documentation
- **Debug Infrastructure**: Comprehensive logging and agent decision visibility

### ⏳ **TODO** (Future Phases)
- Chat system implementation (send_chat_message action and communication phases)
- Agent integration with ADK framework  
- Advanced web features (WebSocket real-time updates)
- Performance monitoring with Langfuse
- Tournament systems and multi-game analytics

### 🎯 **Production Ready**
The Secret AGI system is now fully production-ready with:
- **Complete Game Engine**: 189/189 tests passing with async GameEngine
- **Type Safety**: 0 mypy errors with strict configuration  
- **Code Quality**: Full linting and formatting pipeline
- **Developer Experience**: Complete tooling with Justfile and database commands
- **Async Architecture**: Single consolidated async GameEngine with mandatory persistence
- **Database Persistence**: Complete SQLModel/SQLite integration with Alembic migrations
- **Game Completeness**: Reliable game termination across all player counts
- **Agent Infrastructure**: SimpleOrchestrator, testing pipeline, web interface
- **Detailed Logging**: Turn-by-turn action history with validation status
- **Recovery Systems**: Complete game state recovery and checkpoint functionality

**Ready to support:**
- **Immediate Agent Development**: Complete infrastructure for building and testing agents
- **Game Monitoring**: Web interface with real-time action-by-action visibility
- **Performance Analysis**: Database-backed metrics collection and analysis
- **Game Replay**: Complete historical game state access and reconstruction
- **Multi-Agent Scenarios**: Mixed agent type coordination and testing
- **Production Deployment**: Enterprise-grade persistence and error recovery

## Development Memories

- Always use jj to set commit messages. Never output them as text. Don't add yourself (Claude) to commit message or a a co-author.
- I feel we're complicating our life by adding new features before all tests are passing. Make sure all tests are passing after every time you make changes
- **IMPORTANT**: ALWAYS use jj (Jujutsu) for version control operations, NEVER git. This is a jj repository.
- Don't remove documentation files unless prompted to
- Always write commit message after making changes if it's empty

---
> Source: [wojtyniak/secret-agi](https://github.com/wojtyniak/secret-agi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
