## agent-company

> This is an AI work team project where multiple specialized agents collaborate to build applications. Each agent has a specific role and expertise, working together to create complete software solutions.

# CLAUDE.md - Agent Company Project

## Project Overview

This is an AI work team project where multiple specialized agents collaborate to build applications. Each agent has a specific role and expertise, working together to create complete software solutions.

## Project Structure

```
Agent-Company/
├── agents/              # Individual agent implementations
│   ├── figma_designer/  # Figma design agent
│   └── ...              # Future agents
├── shared/              # Shared utilities and modules
├── config/              # Configuration files
├── tests/               # Test files
└── docs/                # Documentation
```

## Current Agents

### 1. Figma Designer Agent
- **Purpose**: Designs applications and UI/UX mockups on Figma
- **Location**: `agents/figma_designer/`
- **Status**: Integrated with MCP server
- **Integration**: Uses `claude-talk-to-figma-mcp` MCP server for Figma communication
- **Communication**: WebSocket client (`shared/figma_mcp_client.py`) connects to MCP server on port 3055

## Development Guidelines

### Adding New Agents
1. Create a new directory under `agents/`
2. Implement the agent following the base agent structure
3. Add configuration in `config/agents.yaml`
4. Update this file with agent documentation

### Agent Communication
- Agents communicate through shared interfaces and message queues
- Each agent should expose a standard API for interaction
- Use the shared utilities in `shared/` for common functionality

## Technology Stack
- Python 3.9+
- WebSockets (for MCP communication)
- Bun runtime (for MCP server)
- Figma Desktop + Plugin (for Figma Designer agent)

## Getting Started

1. Install dependencies: `pip install -r requirements.txt`
2. For Figma Designer Agent:
   - Start MCP WebSocket server: `cd claude-talk-to-figma-mcp && bun socket`
   - Install Figma plugin (see `agents/figma_designer/SETUP.md`)
   - Configure channel in `config/agents.yaml`
3. Configure agents in `config/agents.yaml`
4. Run individual agents or the orchestrator

## Notes for Claude

- When working on this project, maintain the modular agent structure
- Each agent should be self-contained but use shared utilities
- Follow the existing patterns when adding new agents
- Keep agent-specific logic isolated from shared code

### Figma Designer Agent Specific Notes

- The agent uses the MCP server located in `claude-talk-to-figma-mcp/`
- The MCP server must be running (`bun socket`) before using the agent
- The agent communicates via WebSocket to the MCP server (port 3055)
- The Figma plugin must be installed and running in Figma Desktop
- Channel ID is required - get it from the Figma plugin UI
- All MCP commands are available through the `mcp_command` action or specific action handlers

---
> Source: [tiagorebelo97/Agent-Company](https://github.com/tiagorebelo97/Agent-Company) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
