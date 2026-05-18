## claude-code-mcp

> This project aims to leverage the Model Context Protocol (MCP) to enhance Claude's capabilities through external tools and APIs.

# Claude Code Helper Guide

## Project Architecture Overview

This project aims to leverage the Model Context Protocol (MCP) to enhance Claude's capabilities through external tools and APIs.

Key concepts:
- We use MCP servers from two sources:
  1. **Reference servers** - Located at `$HOME/dev/mcp-servers-repo` (cloned from [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers))
  2. **Local servers** - Located at `$HOME/dev/claude-code/mcp-servers/` (our own implementations)

The reference MCP servers (from `MCP_REPO_PATH`) should be treated as read-only. Our local MCP servers (in `MCP_LOCAL_PATH`) will be our own implementations, some of which will be based on the reference servers but with extensions and improvements.

### Directory Structure
- `docs/` - Reference documentation
- `mcp-servers/` - Our local implementations of MCP servers
- `archive/` - Storage for deprecated or unused code components
- `data/` - Data storage for local servers (e.g., sqlite, memory)
- `scripts/` - Utility scripts for setup and testing
- `src/` - Source code for Python modules and utilities

## MCP Server Configuration

The primary configuration is handled by the `claude-mcp` script at the project root:

```bash
# Script path
/Users/williambrown/dev/claude-code/claude-mcp
```

Key environment variables:
- `CLAUDE_FILESYSTEM_PATH="$HOME/dev"` - Base path for filesystem access
- `MCP_REPO_PATH="$HOME/dev/mcp-servers-repo"` - Reference MCP servers (read-only)
- `MCP_LOCAL_PATH="$HOME/dev/claude-code/mcp-servers"` - Our local MCP server implementations
- `CLAUDE_MEMORY_PATH="$HOME/dev/claude-code/data/memory/memory.json"` - Memory file path
- `CLAUDE_SQLITE_PATH="$HOME/dev/claude-code/data/sqlite/test.db"` - SQLite database path

### MCP Server Registration
The script uses `claude mcp add` commands to register servers with Claude. It uses two types of servers:

1. **Package-based servers** (from npm/pypi):
   ```bash
   # Examples
   claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem $CLAUDE_FILESYSTEM_PATH
   claude mcp add fetch -- uvx mcp-server-fetch
   ```

2. **Local servers** (from our repository):
   ```bash
   # Example
   claude mcp add sqlite -- uv --directory "${MCP_REPO_PATH}/src/sqlite" run mcp-server-sqlite --db-path $CLAUDE_SQLITE_PATH
   ```

### Required Environment Variables
- `BRAVE_API_KEY` - For brave-search server
- `GITHUB_TOKEN` - For GitHub operations (formerly GITHUB_PERSONAL_ACCESS_TOKEN)
- `LINEAR_API_KEY` - For Linear issue tracking
- `SLACK_BOT_TOKEN` and `SLACK_TEAM_ID` - For Slack communication
- `E2B_API_KEY` - For code execution sandbox
- `ALLOWED_PATHS` - For filesystem access control (defaults to project root)

### Important MCP Servers
- **brave-search** - Web search capabilities
- **github** - GitHub repository operations
- **filesystem** - Local file system operations
- **fetch** - Web page fetching
- **memory** - Knowledge graph for persistent memory
- **sqlite** - Database operations
- **slack** - Slack communication
- **linear** - Linear issue tracking integration
- **e2b** - Code execution sandbox
- **research-papers** - Research paper management with Semantic Scholar integration
- **mcp-test** - Testing client for developing and testing other MCP servers

### MCP Server Usage Guidelines

When to use specific MCP servers:

- **brave-search**: Use for general web searches and finding current information.
  - Best for: Finding documentation, articles, tutorials, product information
  - Example: `mcp__brave-search__brave_web_search(query="python type hints tutorial")`

- **github**: Use for interacting with GitHub repositories.
  - Best for: Examining code, creating issues/PRs, searching repos, accessing GitHub content
  - Example: `mcp__github__search_repositories(query="semantic scholar api")`

- **filesystem**: Use for reading and manipulating local files.
  - Best for: Reading, writing, and exploring files in allowed directories
  - Example: `mcp__filesystem__read_file(path="/path/to/file")`

- **fetch**: Use for downloading web content directly.
  - Best for: Reading web articles, documentation, accessing external APIs
  - Example: `mcp__fetch__fetch(url="https://example.com/api/docs")`

- **docker**: Use for running isolated code and managing development environments.
  - Best for: Executing code snippets, running code with dependencies, creating custom environments
  - Run code in ephemeral containers with various languages (with internet access)
  - Register custom Docker templates via tool calls
  - Support for Python, Node.js, Ruby, and more
  - Example: `mcp__docker__docker_run_code(language="python", code="import requests\nprint(requests.get('https://httpbin.org/get').json())", dependencies=["requests"])`
  - Example: `mcp__docker__docker_register_template(name="custom-py", image="python:3.11-slim", description="Custom Python environment")`

- **memory**: Use for persistent knowledge storage across sessions.
  - Best for: Storing user preferences, frequently used commands, project context
  - Example: `mcp__memory__create_entities(...)`

- **sqlite**: Use for structured data storage and querying.
  - Best for: Storing tabular data, performing complex queries, data analysis
  - Example: `mcp__sqlite__read_query(query="SELECT * FROM table")`

- **research-papers**: Use for academic research and paper management.
  - Best for: Searching academic papers, organizing research, tracking citations
  - Available tools: paper search, collections, tagging, citation management
  - Example: Search for papers about machine learning, create collections, add notes

- **mcp-test**: Use for developing and testing MCP servers.
  - Best for: Testing server implementations, calling tools on new servers, viewing logs
  - Example: `mcp__mcp-test__mcp_test_deploy_server(name="my-server", source_path="/path/to/server")`

For optimal results:
1. Choose the most specific server for your task
2. Combine servers when needed (e.g., search with brave-search, then fetch details)
3. Use filesystem for local operations and fetch for remote content
4. Use github for repository-specific operations
5. Consider research-papers for academic literature search instead of generic web search
6. Use mcp-test for testing new MCP servers during development

## Development Guidelines

### Note Tracking
- When the user says "(note this)" in conversation, always add the information to CLAUDE.md in the appropriate section for future reference
- Ensure to organize notes under relevant headings to maintain document structure
- Update existing information when notes provide new guidance or clarifications
- NEVER write environment variable values to notes or files, even in examples

### Package Management & Build
- Python package management: `uv add <package>` (NEVER pip)
- Running Python tools: `uv run <tool>`
- Upgrading packages: `uv add --dev package --upgrade-package package`
- TypeScript build: `npm run build`
- FORBIDDEN: `uv pip install`, `@latest` syntax

### Lint & Test Commands
- Format: `uv run ruff format .`
- Lint: `uv run ruff check .` (fix with `--fix`)
- Type check: `uv run pyright`
- Testing: `uv run pytest`
- Single test: `uv run pytest path/to/test.py::test_function`
- Async testing: use anyio, not asyncio

### Code Style Guidelines
- Type hints required for all code
- Public APIs must have docstrings
- Functions must be small and focused
- 88 character line length maximum
- Follow existing patterns exactly
- Use clean, modular abstractions and proper Python OOP principles
- Explicit None checks for Optional types
- Address problems generally, avoid code duplication
- Comments/names should never mention previous errors
- Names should "make sense" without change history context
- Strive for minimal clean code that's easily maintained and extended
- Regularly search codebase to identify areas where code can be consolidated
- New features require tests
- Bug fixes require regression tests
- Test edge cases and error paths
- Always carefully discuss design choices about major features before proceeding with implementation

### Error Resolution Priority
1. Formatting
2. Type errors
3. Linting

### Line Wrapping
- Strings: use parentheses
- Function calls: multi-line with proper indent
- Imports: split into multiple lines
- Follow Ruff I001 for import sorting

### Git Workflow
- Develop in separate branches, merge to main quickly
- For user-reported fixes: `git commit --trailer "Reported-by:<user>"`
- For GitHub issues: `git commit --trailer "Github-Issue:#<number>"`
- Always use `git add .` and rely on .gitignore to exclude files, rather than manually selecting files
- NEVER push before thoroughly testing the changes
- ALWAYS push to the private repository (`git push private main`) by default
- Only push to the public repository (`git push origin main`) when explicitly requested
- NEVER mention co-authored-by or tools used to create commits/PRs

### GitHub Commit Attribution
- Use git configuration to set Claude as the commit author:
  ```bash
  git config --local user.name "Claude"
  git config --local user.email "noreply@anthropic.com"
  ```
- This ensures commits appear as coming from Claude while using your PAT
- Add co-authored-by trailers in commit messages for proper attribution

### Pull Request Guidelines
- Create detailed messages focusing on high-level description of the problem and solution
- Don't go into code specifics unless it adds clarity
- Add required reviewers according to project guidelines
- NEVER mention tools used to create the PR

## MCP Server Development Guidelines

When developing our own local MCP servers:

1. **Prototype First**:
   - Develop initial versions in the `playground/` directory first
   - Test functionality thoroughly before moving to `mcp-servers/`
   - Use the playground for experimentation and rapid iteration

2. **Reference Implementation First**:
   - Study the original server in `MCP_REPO_PATH`
   - Understand its API, behavior, and implementation details
   - Document the key functions and features

3. **Implementation Strategy**:
   - Start by mimicking the reference server exactly
   - Once basic functionality is working, add extensions
   - Ensure backward compatibility

4. **Code Quality**:
   - Use proper typing (TypeScript/Python type hints)
   - Add comprehensive documentation
   - Implement proper error handling
   - Follow consistent naming conventions
   - Write tests for all functionality

5. **Documentation**:
   - Update README.md for each server
   - Document extensions and differences from reference implementation
   - Provide usage examples

### MCP Server Development Workflow

The complete workflow for developing, testing, and deploying a new MCP server is:

1. **Develop in Playground**:
   - Create or modify server code in the `playground/` directory
   - Follow established patterns from reference implementations
   - Ensure proper error handling and typing

2. **Test with MCP Test Client**:
   - Deploy server to persistent test container using MCP Test Client
   - Run test suites to verify functionality
   - Make fixes and iteratively improve
   - Test interactively with realistic scenarios
   - View logs and error messages for debugging

3. **Validate and Migrate**:
   - Run final validation checks with comprehensive test suites
   - Migrate code to `mcp-servers/` when ready
   - Run tests again to verify deployment

4. **Register with Claude**:
   - Add server to `claude-mcp-local` script
   - Register with `claude mcp add` command
   - Verify registration was successful

Refer to `notes/mcp_test_client_design.md` for detailed information about the testing workflow.

### Using the MCP Test Client

The MCP Test Client provides tools for testing and validating MCP servers during development without needing to restart Claude sessions or formally register servers. It acts as middleware between Claude and the server under test.

#### Key Testing Workflow:

1. **Deploy the server under test**:
   ```bash
   mcp__mcp-test__mcp_test_deploy_server \
     --name "my-server" \
     --source_path "/path/to/server" \
     --env_vars '{"ENV_VAR": "value"}'
   ```

2. **Run tests against the deployed server**:
   ```bash
   mcp__mcp-test__mcp_test_run_tests \
     --server_name "my-server" \
     --test_suite "my-test-suite" 
   ```

3. **Call specific tools for interactive testing**:
   ```bash
   mcp__mcp-test__mcp_test_call_tool \
     --server_name "my-server" \
     --tool_name "tool_name" \
     --arguments '{"param": "value"}'
   ```

4. **View server logs for debugging**:
   ```bash
   mcp__mcp-test__mcp_test_get_logs \
     --server_name "my-server" \
     --lines 100
   ```

5. **Clean up when finished**:
   ```bash
   mcp__mcp-test__mcp_test_stop_server \
     --server_name "my-server"
   ```

#### Creating Test Suites:

Test suites are JSON files placed in `/mcp-servers/mcp-test-client/test/` with the following structure:

```json
{
  "name": "Test Suite Name",
  "description": "Description of the test suite",
  "tests": [
    {
      "name": "Test Name",
      "description": "Test description",
      "tool": "tool_name",
      "input": {
        "param1": "value1",
        "param2": "value2"
      },
      "expected": {
        "type": "contains",
        "value": "expected substring"
      }
    }
  ]
}
```

Expected types include:
- `contains`: Result contains expected value
- `equals`: Result equals expected value
- `nonEmpty`: Result is not empty
- `matches`: Result matches regex pattern

## Server-Specific Information

### Documentation Guidelines
- All design documents should be stored in the `notes/` folder
- Current design documents:
  - Docker MCP: `notes/docker_mcp_design.md` - Focused on isolated code execution environments
  - MCP Test Client: `notes/mcp_test_client_design.md` - Testing framework for MCP servers
  - MCP Agent: `notes/mcp_agent_next_steps.md` - Self-testing capabilities for Claude
- Use clear, descriptive filenames for design documents (e.g., `feature_name_design.md`)

### Docker MCP Server

### MCP Server Implementation Principles
- Strive for consistency in implementation styles across all servers unless there's a compelling reason to do otherwise
- Always follow the same patterns and practices as existing servers when creating new MCP servers
- Study reference implementations thoroughly before starting development
- Ensure compatibility with the current MCP SDK version
- Follow TypeScript/Python best practices appropriate to the language
- Use proper error handling with custom error classes
- Implement comprehensive logging with different log levels
- Create detailed documentation in README.md files
- Write tests for all functionality

### TypeScript MCP Servers
- Use ES modules (set `"type": "module"` in package.json)
- Configure TypeScript for ES modules:
  ```json
  {
    "compilerOptions": {
      "module": "NodeNext",
      "moduleResolution": "NodeNext"
    }
  }
  ```
- Always use `.js` extension in import paths, even for TypeScript files:
  ```typescript
  import { MyType } from './types.js';  // NOT './types' or './types.ts'
  ```
- Initialize server with proper name and version:
  ```typescript
  const server = new Server(
    { name: "server-name", version: "0.1.0" },
    { capabilities: { tools: {} } }
  );
  ```
- Use `setRequestHandler` for handling MCP protocol requests:
  ```typescript
  // For listing tools
  server.setRequestHandler(ListToolsRequestSchema, async () => {
    return {
      tools: [
        {
          name: "tool_name",
          description: "Tool description",
          inputSchema: {
            type: "object",
            properties: { /* properties */ },
            required: [ /* required properties */ ]
          }
        }
      ]
    };
  });
  
  // For handling tool calls
  server.setRequestHandler(CallToolRequestSchema, async (request) => {
    const { name, arguments: args } = request.params;
    
    // Handle the request and return result in proper format
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(result, null, 2)
        }
      ]
    };
  });
  ```
- Implement proper error handling with custom error classes:
  ```typescript
  if (error instanceof CustomError) {
    return { 
      error: { 
        message: error.message, 
        name: error.name 
      } 
    };
  }
  ```
- Connect to stdio transport in an async main function:
  ```typescript
  async function main() {
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.log('Server running on stdio');
  }
  
  main().catch(console.error);
  ```
- Implement structured logging with configurable log levels
- Make sure to export a proper executable with a shebang:
  ```typescript
  #!/usr/bin/env node
  ```
- Build with `npm run build`

### Python MCP Servers
- Follow PEP standards
- Implement proper type hints
- Use async patterns when appropriate
- Document with docstrings
- Package with `pyproject.toml` and uv

## API Usage Examples

### GitHub API Examples
- `github_create_issue(owner="user", repo="repo-name", title="Issue title")`
- `github_search_repositories(query="search terms")`
- `github_get_file_contents(owner="user", repo="repo", path="file.js")`
- `github_create_pull_request(owner="user", repo="repo", title="PR title", head="feature", base="main")`

### Linear API Examples
- `linear_create_issue(title="Task name", teamId="TEAM-123")`
- `linear_search_issues(query="keyword", status="In Progress")`
- `linear_update_issue(id="ABC-123", status="In Progress")`
- `linear_get_user_issues(limit=20)`
- `linear_add_comment(issueId="ABC-123", body="Working on this now")`

### Slack API Examples
- `slack_post_message(channel_id="C0123456789", text="Hello world!")`
- `slack_reply_to_thread(channel_id="C0123456789", thread_ts="1234567890.123456", text="Reply text")`
- `slack_add_reaction(channel_id="C0123456789", timestamp="1234567890.123456", reaction="thumbsup")`
- `slack_get_channel_history(channel_id="C0123456789", limit=10)`

---
> Source: [willccbb/claude-code-mcp](https://github.com/willccbb/claude-code-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
