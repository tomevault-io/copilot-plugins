## task-w-memory-magic

> This rule explains how to use MCP servers with the Memory Bank system for enhanced capabilities.

# Memory Bank MCP Server Integration

Whenever you use this rule, start your message with the following:

"Checking Memory Bank MCP integration..."

This rule describes how to use Model Control Protocol (MCP) servers with the Memory Bank system to enhance AI agent capabilities. MCP servers provide specialized functionality that can improve the effectiveness of the Memory Bank system.

## Available MCP Servers

The Memory Bank system can leverage three primary MCP servers:

1. **Context7 MCP Server**:
   * Provides up-to-date documentation for libraries and technologies
   * Helps maintain accurate technical information in memory bank files

2. **Sequential Thinking MCP Server**:
   * Enhances planning capabilities through structured thought processes
   * Particularly useful for complex decision-making and problem-solving

3. **Supabase MCP Server**:
   * Facilitates database operations and project management
   * Provides tools for database integration and management

## When to Use MCP Servers

### Context7 MCP Server

Use the Context7 MCP server in these scenarios:

* **Updating techContext.md**:
  * When adding new libraries or technologies to the project
  * When verifying technical details about existing dependencies
  * When documenting API usage patterns

* **Updating systemPatterns.md**:
  * When researching best practices for implementation
  * When documenting library-specific patterns or approaches

* **Technology Research**:
  * Before making significant technology decisions
  * When evaluating library options or alternatives

**Example Usage**:
```
@memory-bank/mcp-integration.md I need to update techContext.md with information about React Query's latest features.
```

### Sequential Thinking MCP Server

Use the Sequential Thinking MCP server in these scenarios:

* **Complex Planning**:
  * When planning complex feature implementations
  * When designing system architecture
  * When making significant technical decisions

* **Problem Solving**:
  * When debugging complex issues
  * When working through difficult implementation challenges
  * When evaluating trade-offs between different approaches

* **Updating vision.md or productContext.md**:
  * When thinking through project direction
  * When refining product requirements
  * When prioritizing features or epics

**Example Usage**:
```
@memory-bank/mcp-integration.md I need to use sequential thinking to plan the implementation approach for our new authentication system.
```

### Supabase MCP Server

Use the Supabase MCP server in these scenarios:

* **Database Operations**:
  * When setting up or modifying database schemas
  * When executing migrations or data transformations
  * When querying data for analysis or documentation

* **Project Management**:
  * When creating or managing Supabase projects
  * When working with branches or environments
  * When generating TypeScript types from database schema

* **Edge Functions**:
  * When deploying or managing serverless functions
  * When documenting API endpoints and their functionality

**Example Usage**:
```
@memory-bank/mcp-integration.md I need to create a migration for our new user profile fields and document it in systemPatterns.md.
```

## MCP Server Tools and Usage

### Context7 MCP Tools

The Context7 MCP server provides two main tools:

1. **resolve-library-id**:
   * **Purpose**: Resolves a package/product name to a Context7-compatible library ID
   * **Parameters**:
     * `libraryName`: The name of the library to search for
   * **When to Use**: Before using `get-library-docs` to obtain the correct library ID

2. **get-library-docs**:
   * **Purpose**: Fetches up-to-date documentation for a library
   * **Parameters**:
     * `context7CompatibleLibraryID`: The ID obtained from `resolve-library-id`
     * `tokens` (optional): Maximum number of tokens to retrieve (default: 10000)
     * `topic` (optional): Topic to focus on (e.g., 'hooks', 'routing')
   * **When to Use**: When you need current documentation about a library or framework

**Integration with Memory Bank**:
* Use library documentation to update `techContext.md` with accurate information
* Document specific patterns in `systemPatterns.md` based on library best practices
* Keep technology information current and accurate

### Sequential Thinking MCP Tool

The Sequential Thinking MCP server provides a flexible thinking process:

* **sequentialthinking**:
  * **Purpose**: Enables dynamic and reflective problem-solving through structured thoughts
  * **Key Features**:
    * Can adjust total thoughts as you progress
    * Can question or revise previous thoughts
    * Can branch or backtrack in thinking process
    * Generates and verifies solution hypotheses
  * **When to Use**: For complex problems requiring step-by-step analysis

**Integration with Memory Bank**:
* Use sequential thinking to update the `activeContext.md` with well-reasoned decisions
* Document architectural decisions in `systemPatterns.md` with clear reasoning
* Plan project direction for `vision.md` with thorough consideration of alternatives

### Supabase MCP Tools

The Supabase MCP server provides numerous tools for database and project management:

1. **Project Management Tools**:
   * `list_projects`, `get_project`, `create_project`, `pause_project`, `restore_project`
   * `list_organizations`, `get_organization`

2. **Database Operation Tools**:
   * `list_tables`, `list_extensions`, `list_migrations` 
   * `apply_migration`, `execute_sql`
   * `get_logs`

3. **Project Configuration Tools**:
   * `get_project_url`, `get_anon_key`

4. **Branching Tools**:
   * `create_branch`, `list_branches`, `delete_branch`, `merge_branch`, `reset_branch`, `rebase_branch`

5. **Development Tools**:
   * `generate_typescript_types`

**Integration with Memory Bank**:
* Document database schema and changes in `systemPatterns.md`
* Include API endpoints and authentication details in `techContext.md`
* Track database migrations and their purpose in `state.md`

## MCP Integration Process

When using MCP servers with the Memory Bank system:

1. **Identify Need**:
   * Determine which MCP server is appropriate for the current task
   * Clarify the specific information or capability needed

2. **Execute MCP Request**:
   * Reference this rule to use the correct MCP server and tool
   * Provide clear parameters and context for the request

3. **Update Memory Bank**:
   * Use the information or insights gained to update relevant Memory Bank files
   * Document the use of MCP servers in the updates for future reference

4. **Cross-Reference**:
   * Ensure consistency between MCP-derived information and existing Memory Bank content
   * Update related files as needed for complete coverage

## Best Practices

1. **Use Context7 Regularly**:
   * Keep `techContext.md` updated with the latest library information
   * Reference library-specific patterns in `systemPatterns.md`

2. **Apply Sequential Thinking for Complex Problems**:
   * Use for architecture decisions and planning
   * Document the thought process in `activeContext.md` or `systemPatterns.md`

3. **Leverage Supabase for Database Documentation**:
   * Generate and maintain accurate database schema documentation
   * Keep track of migrations and their impact in `state.md`

4. **Avoid Overreliance**:
   * Use MCP servers as tools to enhance the Memory Bank, not replace it
   * Always update Memory Bank files with MCP-derived information for future sessions

By effectively integrating MCP servers with the Memory Bank system, AI agents can maintain more accurate, current, and comprehensive project context across sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miladnoo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
