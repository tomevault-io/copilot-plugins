## fastmcp-quickstart-20251009-8iit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a FastMCP quickstart project demonstrating the Model Context Protocol (MCP) server implementation using FastMCP 2.0. The repository contains two MCP servers:

1. **echo.py** - Simple echo server demonstrating basic FastMCP capabilities
2. **prompt_enhancer.py** - Production-ready server for improving prompts for Claude Code

## Architecture

**FastMCP Server Pattern**: The codebase follows FastMCP's decorator-based pattern where a single `FastMCP` instance (`mcp`) serves as the server object, and functionality is registered using decorators:

- `@mcp.tool` - Defines callable tools that LLMs can invoke
- `@mcp.resource` - Exposes data resources (static or templated URIs)
- `@mcp.prompt` - Registers reusable prompt templates

The server in `echo.py` demonstrates all three decorator types with echo functionality.

### Prompt Enhancer Server

`prompt_enhancer.py` is a comprehensive MCP server designed to improve prompts for Claude Code:

**Tools (6 functions)**:
- `enhance_prompt` - Main enhancement function with category support (coding/debugging/refactoring/testing)
- `add_technical_context` - Adds language, framework, file paths
- `make_specific` - Transforms vague prompts into specific, actionable ones
- `add_best_practices` - Adds coding standards and constraints
- `analyze_prompt` - Scores and analyzes prompt quality (0-100 scale)
- `compare_prompts` - Side-by-side comparison of original vs enhanced

**Resources (4 endpoints)**:
- `prompts://best-practices` - Comprehensive guide for writing effective prompts
- `prompts://examples/{category}` - Examples for coding, debugging, refactoring, testing
- `prompts://templates/{task_type}` - Ready-to-use templates (create_function, fix_bug, etc.)
- `prompts://anti-patterns` - Common mistakes to avoid

**Prompts (3 templates)**:
- `enhance-for-claude` - Base template for prompt enhancement
- `add-context-template` - Template for adding technical context
- `make-actionable-template` - Template for making prompts concrete

## Development Commands

### Running the Server

**Local development with MCP Inspector** (recommended for testing):
```bash
# Echo server
fastmcp dev echo.py

# Prompt Enhancer server
fastmcp dev prompt_enhancer.py
```
This runs the server via `uv run` subprocess with the MCP Inspector for interactive testing. Note: This command only supports STDIO transport.

**Run server directly in current environment**:
```bash
# Echo server
python echo.py
# or
python3 echo.py

# Prompt Enhancer server
python prompt_enhancer.py
```
By default, runs with STDIO transport.

**Run with HTTP transport** (for web-based access):
```bash
fastmcp run echo.py --transport http --port 8000
# or in code, modify echo.py to include:
# mcp.run(transport="http", host="127.0.0.1", port=8000)
```

**Run with SSE transport** (legacy):
```bash
fastmcp run echo.py --transport sse --port 8000
```

### Testing

**In-memory testing** (fastest, no network/subprocess):
```python
from fastmcp import Client
import asyncio

async def main():
    # Import your server
    from echo import mcp

    # Create in-memory client
    async with Client(mcp) as client:
        # Test ping
        assert await client.ping()

        # List and test tools
        tools = await client.list_tools()
        result = await client.call_tool("echo_tool", {"text": "Hello"})
        print(result)

asyncio.run(main())
```

**Testing with HTTP client**:
```python
from fastmcp import Client
import asyncio

async def main():
    async with Client("http://localhost:8000/mcp/") as client:
        assert await client.ping()
        result = await client.call_tool("echo_tool", {"text": "Hello"})
        print(result)

asyncio.run(main())
```

## Key Concepts

**Transport Mechanisms**: FastMCP supports multiple transports:
- STDIO (default) - For local CLI integration
- Streamable HTTP (modern, default for HTTP) - For web deployments
- SSE (legacy) - Server-Sent Events for web deployments

**Server Lifecycle**: Always use `if __name__ == "__main__":` block with `mcp.run()` for compatibility with various MCP clients.

**Resource URIs**: Resources support both static (`echo://static`) and templated URIs (`echo://{text}`) for dynamic content.

## Deployment

This repository is designed for FastMCP Cloud deployment:
1. Create account at http://fastmcp.cloud/signup
2. Connect GitHub account
3. Select "Clone our template" for automatic deployment

## Using Prompt Enhancer

### Example Usage with Claude Code

After installing the Prompt Enhancer server in Claude Code:

```bash
fastmcp install claude-code prompt_enhancer.py --server-name "Prompt Enhancer"
```

You can use it to improve your prompts:

1. **Enhance a vague prompt**:
   - "Use enhance_prompt tool to improve: 'fix the bug' for category 'debugging'"

2. **Analyze prompt quality**:
   - "Use analyze_prompt to score: 'Create a function to process users'"

3. **Get best practices**:
   - "Read the prompts://best-practices resource"

4. **See examples**:
   - "Show me examples from prompts://examples/coding"

5. **Get templates**:
   - "Get the template from prompts://templates/create_function"

### Common Workflows

**Before asking Claude Code to write code**:
1. Use `analyze_prompt` to check your prompt quality
2. Use `enhance_prompt` to get suggestions
3. Read relevant examples from resources
4. Use the enhanced prompt with Claude Code

**For testing**: See `HOW_TO_TEST.md` for detailed testing instructions and examples.

## Documentation

- FastMCP Documentation: https://gofastmcp.com/
- MCP Protocol: https://modelcontextprotocol.io/
- FastMCP has MCP server tools available via Claude Code's MCP integration for searching docs
- Prompt Enhancer testing guide: See `HOW_TO_TEST.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ilyanosovsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ilyanosovsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
