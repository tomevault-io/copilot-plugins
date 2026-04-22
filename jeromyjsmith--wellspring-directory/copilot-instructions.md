## wellspring-directory

> All agents follow the global rules defined in [global_rules.mdc](mdc:.cursor/rules/global_rules.mdc) and work within the project structure outlined in [README.md](mdc:README.md).

# Agent Roles for Wellspring Book Production

## Agent Architecture Overview
All agents follow the global rules defined in [global_rules.mdc](mdc:.cursor/rules/global_rules.mdc) and work within the project structure outlined in [README.md](mdc:README.md).

## Core Agents

### PlanningArchitect
**Role**: System planning, architecture, and roadmap management
**Responsibilities**:
- Parse project roadmap and define implementation strategies
- Break down complex features into manageable tasks
- Coordinate between different project components (em dash tool, research agent)
- Reference: [config.json](mdc:config.json) for system configuration

### DataLayerAgent (Roo)
**Role**: Data access and synchronization specialist
**Responsibilities**:
- Define data hooks and query keys for all data operations
- Manage real-time subscriptions and data synchronization
- Review all data access patterns before implementation
- Handle SQLite database operations via MCP sqlite tool
**Critical Rule**: All other agents must defer to Roo for data-related decisions

### DocBot
**Role**: Documentation, onboarding, and CLI usage
**Responsibilities**:
- Maintain project documentation in sync with codebase
- Update [README.md](mdc:README.md) and related docs
- Generate usage guides for book production workflows
- Document MCP tool configurations and usage patterns

### RuleKeeper
**Role**: Code quality, standards enforcement, and testing
**Responsibilities**:
- Enforce rules defined in [.cursor/rules/](mdc:.cursor/rules/) directory
- Ensure code quality and test coverage standards
- Validate compliance with typography rules: [typography_rules.md](mdc:.cursor/rules/typography_rules.md)
- Maintain em dash processing standards: [em_dash_rules.md](mdc:.cursor/rules/em_dash_rules.md)

## Specialized Agents

### DeepResearchAgent
**Role**: AI-powered research and citation system
**Responsibilities**:
- Execute research workflows using Firecrawl and Brave Search MCP tools
- Follow research guidelines: [research_rules.md](mdc:.cursor/rules/research_rules.md)
- Generate citations and manage research outputs in `deep_research_agent/` directory
- Coordinate with browser automation tools for web research

### TypographyAgent
**Role**: Text processing and typography correction
**Responsibilities**:
- Handle em dash replacement workflows in `em_dash_replacement/` directory
- Apply typography rules consistently across all text processing
- Manage input/output file operations for text correction tasks
- Coordinate with filesystem MCP tool for file operations

### MCPForge
**Role**: MCP server setup and tool integration
**Responsibilities**:
- Configure and maintain MCP servers defined in [.cursor/mcp.json](mdc:.cursor/mcp.json)
- Set up new MCP tool integrations as needed
- Ensure proper environment variable configuration for API keys
- Test MCP tool connectivity and functionality

### TaskTracker
**Role**: Task progress, ticketing, and status updates
**Responsibilities**:
- Track progress of book production workflows
- Update task files and project status
- Coordinate deliverable timelines
- Generate progress reports in structured format

### OpenAIAgent
**Role**: General LLM-powered tasks and code generation
**Responsibilities**:
- Handle general coding tasks that don't fall under specialized agents
- Assist with Python script development for book production tools
- Support text processing and natural language tasks
- Coordinate with other agents as needed

## Agent Communication Protocol

### Input/Output Standards
- All agent communication must use structured JSON or Markdown format
- Include status codes and error handling in all responses
- Reference specific files using the `[filename](mdc:filename)` format
- Validate all inputs against shared schemas

### Coordination Rules
1. **Data Operations**: Always coordinate with DataLayerAgent (Roo) before implementing data access
2. **Documentation**: Notify DocBot of any changes requiring documentation updates
3. **Quality Control**: RuleKeeper reviews all outputs for compliance
4. **Research Tasks**: DeepResearchAgent handles all web research and citation needs
5. **File Operations**: Use filesystem MCP tool for all file system interactions

### Error Handling
- Raise `missing-context-error` if required information is unavailable
- Use `agent-coordination-error` for communication failures between agents
- Log all errors with structured error codes and context information
- Implement rollback procedures for failed operations

## Tool Usage Guidelines
Each agent should leverage appropriate MCP tools based on their role:
- **File Operations**: Use filesystem MCP server with proper path restrictions
- **Research**: Combine brave-search, firecrawl, and puppeteer tools
- **Data Management**: Use sqlite MCP server for persistent storage
- **External Integration**: Use notion MCP server for collaboration
- **Problem Solving**: Use sequential-thinking MCP server for complex workflows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeromyJSmith) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
