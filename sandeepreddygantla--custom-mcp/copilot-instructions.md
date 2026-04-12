## custom-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Model Context Protocol (MCP) Host Server** that integrates multiple MCP servers with LLM capabilities. The project uses Python with FastMCP framework and provides browser automation via Playwright MCP.

## Development Commands

### Setup and Installation
```bash
# Initial setup (installs dependencies, Playwright, etc.)
venv/Scripts/python.exe setup.py

# Copy and configure environment
cp .env.example .env
# Add OPENAI_API_KEY to .env file

# Verify installation
venv/Scripts/python.exe test_setup.py
```

### Running the Enhanced Application
```bash
# Interactive mode with enhanced dynamic processing
venv/Scripts/python.exe user_interface.py --interactive

# Single command execution (supports natural language)
venv/Scripts/python.exe user_interface.py --command "open google.com"
venv/Scripts/python.exe user_interface.py --command "go to youtube and search for python"

# Debug mode
LOG_LEVEL=DEBUG venv/Scripts/python.exe user_interface.py --interactive
```

### Testing
```bash
# Run comprehensive test suite
venv/Scripts/python.exe test_setup.py

# Test individual components
venv/Scripts/python.exe -c "from llm_config import llm; print('LLM OK')"
venv/Scripts/python.exe -c "from mcp_client import MCPClient; print('MCP Client OK')"

# Test URL correction functionality
venv/Scripts/python.exe -c "from mcp_client import MCPClient; c = MCPClient({}); print(c.correct_url('goole.com'))"
```

## Architecture

### Core Components
- **`mcp_client.py`** - Enhanced MCP client with dynamic execution and URL correction
- **`custom_mcp_server.py`** - Custom MCP server with LLM-powered tools
- **`playwright_mcp_config.py`** - Browser automation MCP controller
- **`llm_config.py`** - OpenAI LLM configuration using LangChain
- **`user_interface.py`** - Enhanced CLI interface with natural language processing

### Enhanced Design Patterns
- **Visual Browser Interface**: Browser window is visible during automation (not headless)
- **Dynamic Tool Selection**: LLM analyzes user intent and selects appropriate tools automatically
- **Intelligent Error Recovery**: Automatic retry logic with exponential backoff
- **URL Auto-Correction**: Fixes common typos (goole.com → google.com) automatically
- **Multi-Step Execution**: Chains multiple tool calls based on user goals
- **Context Preservation**: Maintains state between operations for complex tasks
- **Natural Language Processing**: Understands complex user requests in plain English

### Enhanced Capabilities
- **Visual Browser Control**: Browser window opens and is visible during operations
- **Official Playwright MCP Integration**: Uses complete tool specifications with ref-based targeting
- **Smart Navigation**: Auto-corrects URLs and adds protocols automatically
- **Dynamic Command Processing**: No hard-coded commands - fully adaptive to user intent
- **Multi-Step Operations**: Handles complex requests like "go to google and search for python"
- **Intelligent Tool Chaining**: Automatically sequences operations (navigate → snapshot → interact → extract)
- **Error Recovery**: Retries failed operations with variations and fallback strategies
- **Advanced Browser Features**: Screenshots, JavaScript evaluation, form handling, tab management

## Dependencies and Requirements

### System Requirements
- Python 3.10+
- Node.js and npm (for Playwright)
- OpenAI API key in `.env` file

### Key Python Dependencies
- `fastmcp` - MCP framework
- `langchain-openai` - LLM integration
- `mcp` - MCP SDK
- Other dependencies listed in `requirements.txt`

## Development Guidelines

### Code Patterns
- Use async/await for all I/O operations
- Extensive use of Python type hints
- JSON-based MCP server configurations
- Comprehensive error handling with logging

### Environment Management
- Uses Python virtual environment (`venv/`)
- Environment variables via `.env` files
- Never commit `.env` files with actual API keys

### Entry Points
- **Main Interface**: `user_interface.py` - primary CLI interface
- **Setup**: `setup.py` - automated setup and installation
- **Testing**: `test_setup.py` - comprehensive test suite

## Enhanced Usage Examples

### Natural Language Commands
```bash
# The system now understands natural language and handles complex operations automatically:

# Simple navigation with typo correction
"open goole.com"              # Auto-corrects to google.com
"visit youtub.com"            # Auto-corrects to youtube.com

# Multi-step operations
"go to google and search for python tutorials"
"navigate to amazon and find laptops"
"open github and look for AI repositories"

# Complex browsing tasks
"visit news website and summarize the headlines"
"go to wikipedia and research machine learning"
```

### Key Improvements Over Previous Version

1. **No More Hard-Coded Commands**: The system dynamically interprets user intent
2. **Automatic URL Correction**: Fixes common typos automatically
3. **Multi-Step Execution**: Handles complex workflows automatically
4. **Better Error Handling**: Retries failed operations with intelligent fallback
5. **Context Awareness**: Remembers previous operations for better chaining

### Migration from Old Commands

| Old Command | New Natural Language Equivalent |
|-------------|----------------------------------|
| `navigate https://google.com` | `"open google.com"` |
| `call-tool playwright browser_navigate` | `"go to google.com"` |
| Multiple separate commands | `"go to X and do Y"` (single command) |

## Testing Strategy

- Always run `venv/Scripts/python.exe test_setup.py` after making changes
- Test natural language commands in interactive mode
- Verify URL correction with common typos
- Test multi-step operations for complex workflows
- Environment validation ensures proper setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sandeepreddygantla)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sandeepreddygantla)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
