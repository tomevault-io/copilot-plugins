## notes

> Write my damn notes!


# MCP Server Project Notes

## Project Structure
```
.
├── .cursor
│   └── rules
│       ├── DONE.mdc
│       ├── WIP.mdc
│       └── notes.mdc
├── reference
│   ├── repomix-mcp-python-sdk.xml
│   └── repomix-openai-agents-python-docs.xml
├── src
│   └── agents_mcp_server
│       ├── __init__.py
│       ├── __main__.py
│       ├── cli.py
│       └── server.py
├── pyproject.toml
└── README.md
```

## Key Technologies
- MCP (Model Context Protocol): Protocol for connecting AI models with external tools and resources
- OpenAI Agents SDK: Framework for creating agents that can use tools
- uv: Package manager for Python (similar to poetry but better)

## Development Guidelines
- Use `uv add <pkg>` to add dependencies instead of directly modifying pyproject.toml
- Use `uv run <cmd-name>` to run CLI commands
- Avoid specifying version numbers when adding dependencies

## OpenAI Agents SDK Patterns
- [best-practice] Specialized agents are more effective for specific tasks than generalist agents
- [best-practice] The Agent.as_tool() method lets you convert agents into tools for other agents to use
- [best-practice] Always use trace() to improve observability of agent execution
- [design] The "agents-as-tools" pattern allows creating an orchestrator agent that delegates to specialist agents
- [gotcha] Be careful with agent instructions - they should be very clear and specific to get the best results
- [gotcha] Each agent instance maintains its own state, so for stateful tools, create a new agent for each request
- [gotcha] FileSearchTool requires a vector_store_ids parameter during initialization - this can't be null or empty
- [gotcha] ComputerTool requires a computer parameter (implementing AsyncComputer) during initialization
- [design] For tools that require client-provided parameters (like FileSearchTool) or implementation details (like ComputerTool), create the agent dynamically at request time rather than pre-initializing it
- [best-practice] Implement a simple placeholder for required interfaces when real implementation isn't available (e.g., SimpleAsyncComputer)

## AsyncComputer Implementation Insights
- [implementation] AsyncComputer is an abstract base class requiring methods for browser/desktop interaction
- [implementation] Required properties: environment (browser/desktop), dimensions (screen size)
- [implementation] Required methods: screenshot, click, double_click, scroll, type, wait, move, keypress, drag
- [best-practice] Maintain state in your AsyncComputer implementation (cursor position, current page, etc.)
- [best-practice] Implement robust command handling for high-level user instructions
- [best-practice] For testing/development, use proper simulation with meaningful outputs rather than empty implementations
- [tip] Real-world implementations can use Playwright (browser) or PyAutoGUI/similar (desktop)
- [design] LocalPlaywrightComputer in the OpenAI docs shows how to implement a real browser-based AsyncComputer

## Agent-Based Design Options
1. **Multi-capable single agent**: A single agent with access to multiple tools (simpler, but less specialized)
2. **Specialized agents with orchestrator**: Multiple agents each specialized in using one tool, with an orchestrator agent that delegates tasks (more complex, but more specialized and effective)

## Tool Requirements
- **WebSearchTool**: No required parameters, but can accept optional `user_location`
- **FileSearchTool**: Requires `vector_store_ids` (list of strings identifying vector stores)
- **ComputerTool**: Requires `computer` (an implementation of AsyncComputer interface)

## Implementation Plan
1. Set up the basic project structure ✅
2. Implement the MCP server using FastMCP ✅
3. Implement OpenAI agent tools integration ✅
4. Implement agent-based approach ✅
5. Add CLI for running and installing the server ✅
6. Create documentation and examples ✅
7. Add tests and improve error handling 🔄
8. Enhance AsyncComputer implementation ✅

---
> Source: [lroolle/openai-agents-mcp-server](https://github.com/lroolle/openai-agents-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
