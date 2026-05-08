## architecture

> Architectural rules and boundaries for SGR Agent Core


# Architectural Rules for SGR Agent Core

## General Architecture

The project uses **modular architecture with clear separation of concerns**. The framework implements Schema-Guided Reasoning (SGR) for building intelligent research agents.

## Core Architecture Layers

### Layer 1: Base Classes (No Dependencies)
- `BaseAgent` - Base class for all agents
- `BaseTool` - Base class for all tools (Pydantic models)
- `MCPBaseTool` - Base class for MCP-integrated tools
- `models.py` - Core data models (AgentContext, AgentStatesEnum, etc.)

### Layer 2: Configuration and Registry
- `agent_config.py` - GlobalConfig singleton
- `agent_definition.py` - AgentDefinition, AgentConfig, LLMConfig, etc.
- `services/registry.py` - AgentRegistry, ToolRegistry
- `services/prompt_loader.py` - PromptLoader service

### Layer 3: Factory and Services
- `agent_factory.py` - AgentFactory for creating agents
- `services/mcp_service.py` - MCP2ToolConverter for MCP tool integration
- `services/tavily_search.py` - Tavily search service
- `next_step_tool.py` - NextStepToolsBuilder, NextStepToolStub, DiscriminantToolMixin

### Layer 4: Agent Implementations
- `agents/sgr_agent.py` - SGRAgent (Structured Output)
- `agents/tool_calling_agent.py` - ToolCallingAgent (Function Calling)
- `agents/sgr_tool_calling_agent.py` - SGRToolCallingAgent (Hybrid)

### Layer 5: Tools
- `tools/reasoning_tool.py` - ReasoningTool
- `tools/clarification_tool.py` - ClarificationTool
- `tools/web_search_tool.py` - WebSearchTool
- `tools/extract_page_content_tool.py` - ExtractPageContentTool
- `tools/generate_plan_tool.py` - GeneratePlanTool
- `tools/adapt_plan_tool.py` - AdaptPlanTool
- `tools/create_report_tool.py` - CreateReportTool
- `tools/final_answer_tool.py` - FinalAnswerTool

### Layer 6: Server and API
- `server/app.py` - FastAPI application
- `server/endpoints.py` - API endpoints
- `server/models.py` - API request/response models
- `server/settings.py` - Server settings
- `stream.py` - OpenAIStreamingGenerator

## Agent Execution Cycle

All agents follow a three-phase cycle implemented in `BaseAgent._execution_step()`:

```python
while agent._context.state not in FINISH_STATES:
    reasoning = await agent._reasoning_phase()
    action_tool = await agent._select_action_phase(reasoning)
    await agent._action_phase(action_tool)
```

The main `execute()` method runs this cycle until agent reaches a finish state.

### Phase 1: Reasoning Phase (`_reasoning_phase()`)
- Agent analyzes current context via `_prepare_context()`
- Decides on next action using LLM
- Implementation varies by agent type:
  - **SGRAgent**: Uses structured output with `NextStepToolStub` (reasoning + tool selection)
  - **ToolCallingAgent**: Returns `None` (no explicit reasoning)
  - **SGRToolCallingAgent**: Uses Function Calling to get `ReasoningTool` result, then executes it
- Returns `ReasoningTool` or `NextStepToolStub` (or `None` for ToolCallingAgent)

### Phase 2: Select Action Phase (`_select_action_phase()`)
- Selects appropriate tool based on reasoning
- Implementation varies by agent type:
  - **SGRAgent**: Extracts tool from `reasoning.function` field
  - **ToolCallingAgent**: Uses Function Calling with `tool_choice="required"`
  - **SGRToolCallingAgent**: Uses Function Calling with `tool_choice="required"` (handles text response edge case)
- Returns `BaseTool` instance ready for execution

### Phase 3: Action Phase (`_action_phase()`)
- Executes selected tool: `result = await tool(context, config)`
- Updates conversation history with tool call and result
- Updates agent context (state, iteration, sources, etc.)
- Logs tool execution
- Handles special cases (e.g., `ClarificationTool` pauses execution)

## Module Rules

### Agents (`sgr_agent_core/agents/`)
- Must inherit from `BaseAgent`
- Must implement `_reasoning_phase()`, `_select_action_phase()`, `_action_phase()`
- Can override `_execution_step()` to customize execution cycle
- Can override `_prepare_context()` and `_prepare_tools()` for customization
- Automatically registered in `AgentRegistry` via `AgentRegistryMixin`
- Must set `name` class variable (used for registration)

### Tools (`sgr_agent_core/tools/`)
- Must inherit from `BaseTool` or `MCPBaseTool`
- Must be Pydantic models (inherit from `BaseModel`)
- Must implement `__call__(context: AgentContext, config: AgentConfig, **kwargs) -> str`
- Must set `tool_name` and `description` class variables (or use defaults)
- Automatically registered in `ToolRegistry` via `ToolRegistryMixin`
- Return string or JSON string from `__call__()`
- Can be used in `NextStepToolsBuilder` for structured output

### Services (`sgr_agent_core/services/`)
- Stateless utility classes
- Provide shared functionality (prompts, MCP, search)
- No direct dependencies on agents or tools

### Configuration (`agent_config.py`, `agent_definition.py`)
- Hierarchical configuration: GlobalConfig → AgentDefinition → AgentConfig
- Supports YAML loading via `GlobalConfig.from_yaml()` and `definitions_from_yaml()`
- Supports environment variables via `pydantic-settings` (prefix `SGR__`)
- Automatic inheritance and override of settings
- `AgentDefinition` inherits from `AgentConfig` (has all config fields + name, base_class, tools)
- Config classes use `extra="allow"` to support custom fields

## Design Principles

1. **One class at a time**: When implementing new functionality, create one class at a time, starting from lower layers
2. **Minimal dependencies**: Each module should depend only on lower layer modules
3. **Type hints**: Use type hints for all functions and methods
4. **Async-first**: All I/O operations must be async
5. **Registry pattern**: Use registries for automatic discovery of agents and tools
6. **Configuration over code**: Prefer YAML configuration over hardcoded values

## References

@docs/en/framework/main-concepts.md
@docs/en/framework/agents.md
@README.md

---
> Source: [vamplabAI/sgr-agent-core](https://github.com/vamplabAI/sgr-agent-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
