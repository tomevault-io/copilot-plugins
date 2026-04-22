## mcp-advanced-multi-agent-ecosystem

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Advanced Multi-Agent Ecosystem - a production-grade Model Context Protocol (MCP) server ecosystem with multiple independent MCP servers for AI-assisted development workflows.

## Build & Development Commands

### Full Setup (first-time)
```bash
./scripts/setup.sh  # Requires python3.12, node, npm
```

### Install/Rebuild All Servers
```bash
./scripts/install-mcp-servers.sh
```

### Verify Installation
```bash
./scripts/test-installation.sh
```

### Individual MCP Servers

**Context Persistence (Python)**
```bash
cd src/mcp-servers/context-persistence
pip install -e .
python3 -m context_persistence.server  # Run standalone
```

**TypeScript Servers (task-orchestrator, search-aggregator, agent-swarm, skills-manager)**
```bash
cd src/mcp-servers/<server-name>
npm install && npm run build
node dist/index.js  # Run standalone
```

**Agent Swarm (Development Mode)**
```bash
cd src/mcp-servers/agent-swarm
npm run dev   # Uses tsx for hot-reload
npm test      # Run tests
npm run lint  # Lint code
```

### Running Tests
```bash
# Agent Swarm
cd src/mcp-servers/agent-swarm
npm test                    # All tests
npm run test:watch          # Watch mode
npm run test:coverage       # With coverage

# Other TypeScript servers
cd src/mcp-servers/<server>
npm test
```

## Architecture

### MCP Server Stack

```
┌─────────────────────────────────────────────────────────────┐
│                     agent-swarm (TypeScript)                │
│  Central orchestration layer with SPARC workflows and       │
│  Boomerang task delegation. Routes to other servers.        │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐   ┌─────────────────┐   ┌────────────────┐
│task-orchestrator│ │search-aggregator │ │context-persistence│
│  (TypeScript) │   │   (TypeScript)  │   │    (Python)    │
│               │   │                 │   │                │
│ - Task CRUD   │   │ - Multi-source  │   │ - SQLite +     │
│ - DAG deps    │   │   search routing│   │   Qdrant       │
│ - Code exec   │   │ - Perplexity,   │   │ - Semantic     │
│ - Git linking │   │   Brave, Google │   │   search       │
│ - Analysis    │   │ - Result caching│   │ - Embeddings   │
└───────────────┘   └─────────────────┘   └────────────────┘
        │                     │                     │
        └─────────────────────┴─────────────────────┘
                              │
                      ┌───────┴───────┐
                      ▼               ▼
              ┌─────────────┐  ┌─────────────┐
              │skills-manager│ │github-oauth │
              │ (TypeScript)│  │ (Node.js)   │
              └─────────────┘  └─────────────┘
```

### Key Components

**agent-swarm** (`src/mcp-servers/agent-swarm/`)
- 7 specialized agent types: research, architect, implementation, testing, review, documentation, debugger
- SPARC workflow manager: Specification → Pseudocode → Architecture → Refinement → Completion
- Boomerang task delegation for iterative refinement
- Routes requests to appropriate downstream MCP servers

**task-orchestrator** (`src/mcp-servers/task-orchestrator/`)
- sql.js for pure JavaScript SQLite (no native compilation)
- Multi-language code execution (Python, JavaScript, Bash, SQL)
- Code quality analysis and security scanning
- DAG-based task dependency tracking
- Git commit linking

**context-persistence** (`src/mcp-servers/context-persistence/`)
- SQLAlchemy async + aiosqlite for conversations
- Qdrant local for vector embeddings
- sentence-transformers for semantic search
- Falls back to hash-based embeddings if model unavailable

**search-aggregator** (`src/mcp-servers/search-aggregator/`)
- Aggregates Perplexity, Brave, Google search providers
- Local caching with sql.js
- API key configuration via environment variables

### Storage Locations

Runtime data is stored in `~/.mcp/`:
```
~/.mcp/
├── context/
│   ├── db/          # SQLite conversation data
│   └── qdrant/      # Vector embeddings
├── tasks/           # Task orchestrator database
├── cache/
│   ├── search/      # Search result cache
│   └── code/        # Code execution temp files
├── logs/            # Server logs
├── skills/          # Skills database
└── agents/          # Agent swarm database
```

Fallback storage in `.mcp-local/` when `~/.mcp` isn't writable.

### Configuration

MCP client configuration template: `configs/roo_mcp_config.json`

Roo Code settings: `~/Library/Application Support/Code/User/globalStorage/rooveterinaryinc.roo-cline/settings/mcp_settings.json`

Environment variables for search-aggregator:
- `PERPLEXITY_API_KEY`
- `BRAVE_API_KEY`
- `GOOGLE_API_KEY` / `GOOGLE_CX`

### External MCP Servers (Git Submodules)

External MCPs are managed as **git submodules** in `src/mcp-servers/external/`:

| Server | Type | Purpose | Source |
|--------|------|---------|--------|
| **context7** | TypeScript | Documentation lookup | [Upstash](https://github.com/upstash/context7) |
| **mcp-code-checker** | Python | Code quality (pylint, pytest, mypy) | [MarcusJellinghaus](https://github.com/MarcusJellinghaus/mcp-code-checker) |

**First-time setup (submodules are initialized automatically):**
```bash
./scripts/setup.sh
# Or manually:
git submodule update --init --recursive
./scripts/setup-external-mcps.sh
```

**Cloning this repository:**
```bash
git clone --recursive https://github.com/gr3enarr0w/MCP_Advanced_Multi_Agent_Ecosystem.git
# Or if already cloned:
git submodule update --init --recursive
```

**Routing keywords** (in `delegate` tool):
- `doc/manual/spec/documentation/reference` → context7 (`resolve-library-id`, `get-library-docs`)
- `lint/pylint/pytest/mypy/type check/code quality` → mcp-code-checker (`run_all_checks`, etc.)
- `search/web/lookup/find` → search-aggregator (`search`)
- Default → task-orchestrator (`create_task`)

External servers are internal to agent-swarm. Roo/Claude only needs agent-swarm configured.

Verify external MCPs: On startup, agent-swarm logs `External MCP availability: {"context7":true,"mcp-code-checker":true}`

## Code Style

- **TypeScript**: Follow existing tsconfig/eslint patterns, explicit types, keep entry modules lean
- **Python**: PEP 8, 4-space indentation, side effects behind `if __name__ == "__main__"`
- **Naming**: kebab-case folders, camelCase/PascalCase exports
- **Commits**: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)

## Key Type Definitions

Agent types (agent-swarm): `src/mcp-servers/agent-swarm/src/types/agents.ts`
- Agent, Task, AgentMessage, AgentType, TaskStatus, MessageType

Task status enum (task-orchestrator): pending, in_progress, blocked, completed

## MCP Tool Reference

### agent-swarm Tools
- `list_agents` - List agents with optional status/type filters
- `delegate_task` - Delegate to specialized agent by type
- `execute_sparc_workflow` - Run full SPARC workflow
- `send_boomerang_task` - Return task for refinement
- `delegate` - Auto-route goal to appropriate server
- `route_plan` - Preview routing decision without execution

### task-orchestrator Tools
- `create_task`, `update_task_status`, `get_task`, `list_tasks`, `delete_task`
- `execute_code`, `execute_python_code`, `execute_javascript_code`, `execute_bash_command`
- `analyze_code_quality`, `scan_security`, `get_code_metrics`
- `get_task_graph` - Dependency graph (JSON or Mermaid)
- `link_git_commit`, `get_recent_commits`

### context-persistence Tools
- `save_conversation`, `load_conversation_history`
- `search_similar_conversations` - Semantic search
- `save_decision`, `get_conversation_stats`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gr3enarr0w) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
