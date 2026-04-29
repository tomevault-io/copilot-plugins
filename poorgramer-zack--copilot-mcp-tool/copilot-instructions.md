## copilot-mcp-tool

> This extension integrates GitHub Copilot CLI with Gemini CLI through the Model Context Protocol (MCP), providing a comprehensive set of AI-powered coding tools.

# GitHub Copilot MCP Server - Gemini CLI Extension

This extension integrates GitHub Copilot CLI with Gemini CLI through the Model Context Protocol (MCP), providing a comprehensive set of AI-powered coding tools.

## Available Tools

### Core Tools

- **`ask-copilot`**: General-purpose coding assistant for help, debugging, and architecture questions.
  - `prompt` (required): Your question or request
  - `context` (optional): Additional context (code, error messages, etc.)
  - `model` (optional): AI model to use (default: claude-sonnet-4.6)
  - `allowAllTools` (optional): Allow all tools (default: true)

- **`copilot-explain`**: Get detailed explanations of code snippets.
  - `code` (required): The code to explain
  - `model` (optional): AI model to use

- **`copilot-suggest`**: Get CLI command suggestions for shell, git, or GitHub operations.
  - `task` (required): Natural language description of what you want to do
  - `model` (optional): AI model to use

- **`copilot-debug`**: Debug code errors and get fix suggestions.
  - `code` (required): The buggy code
  - `error` (required): The error message
  - `context` (optional): Additional context

- **`copilot-refactor`**: Get refactoring suggestions for code improvement.
  - `code` (required): The code to refactor
  - `goal` (optional): Specific refactoring goal (e.g., "improve performance", "add error handling")

- **`copilot-review`**: Get code review feedback with focus areas.
  - `code` (required): The code to review
  - `focusAreas` (optional): Array of focus areas (e.g., ["security", "performance"])

- **`copilot-test-generate`**: Generate unit tests for your code.
  - `code` (required): The code to generate tests for
  - `framework` (optional): Testing framework (e.g., "jest", "pytest", "mocha")

### Session Management

- **`copilot-session-start`**: Start a new conversation session for context tracking.

- **`copilot-session-history`**: Get conversation history for a session.
  - `sessionId` (required): The session ID to retrieve history for

## Available Resources

- **`copilot://session/{sessionId}/history`**: Access conversation history for a specific session.
- **`copilot://sessions`**: List all active sessions.

## Supported AI Models

You can specify any of these models for tools that accept a `model` parameter:

| Model | Description |
|-------|-------------|
| `auto` | Auto model selection |
| `claude-sonnet-4.6` | Claude Sonnet 4.6 (default) |
| `claude-sonnet-4.5` | Claude Sonnet 4.5 |
| `claude-haiku-4.5` | Claude Haiku 4.5 (fastest) |
| `claude-opus-4.7` | Claude Opus 4.7 (most capable Claude) |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `gpt-5.4` | GPT 5.4 (default GPT preference) |
| `gpt-5.3-codex` | GPT 5.3 Codex |
| `gpt-5.2-codex` | GPT 5.2 Codex |
| `gpt-5.2` | GPT 5.2 |
| `gpt-5.4-mini` | GPT 5.4 mini |
| `gpt-4.1` | GPT 4.1 (unlimited usage) |
| `gpt-5-mini` | GPT 5 Mini (unlimited usage) |

> **Tip**: `gpt-5-mini` and `gpt-4.1` have unlimited usage for GitHub Copilot Pro subscribers.

## Installation

### Option 1: Using Gemini CLI command (Recommended)

```bash
gemini mcp add copilot -- npx -y @aykahshi/copilot-mcp-server
```

### Option 2: Copy extension files

Copy the extension files to your Gemini CLI extensions directory:

```bash
# Create extension directory
mkdir -p ~/.gemini/extensions/copilot-mcp

# Copy files
cp gemini-extension.json GEMINI.md ~/.gemini/extensions/copilot-mcp/
```

After installation, restart your Gemini CLI session for the changes to take effect.

## Prerequisites

Before using this extension, ensure you have:

1. **Node.js >= 22.0.0** installed
2. **GitHub Copilot CLI** installed and authenticated:
   ```bash
   npm install -g @github/copilot
   copilot /login
   ```

## Usage Examples

### Ask a coding question
```
Use ask-copilot with prompt="How do I implement a debounce function in TypeScript?"
```

### Explain code
```
Use copilot-explain with code="const memoize = fn => { const cache = new Map(); return (...args) => { const key = JSON.stringify(args); return cache.has(key) ? cache.get(key) : cache.set(key, fn(...args)).get(key); }; };"
```

### Debug an error
```
Use copilot-debug with code="function sum(arr) { return arr.reduce((a) => a+b, 0); }" and error="ReferenceError: b is not defined"
```

### Generate tests
```
Use copilot-test-generate with code="function isPrime(n) { return n > 1 && ![...Array(n).keys()].slice(2).some(i => n % i === 0); }" and framework="jest"
```

### Get command suggestions
```
Use copilot-suggest with task="Find all TypeScript files modified in the last 7 days"
```

### Code review
```
Use copilot-review with code="..." and focusAreas=["security", "performance"]
```

### Use a specific model
```
Use ask-copilot with model="claude-opus-4.7" and prompt="Design a microservices architecture for an e-commerce platform"
```

## Troubleshooting

### "copilot command not found"
```bash
npm install -g @github/copilot
copilot --version
```

### "Not authenticated"
```bash
copilot /login
```

### Server connection issues
Verify the MCP server is running:
```bash
gemini mcp list
```

## Related Projects

- [copilot-mcp-tool](https://github.com/Poorgramer-Zack/copilot-mcp-tool) - Main repository
- [copilot-flow plugin](plugins/copilot-flow/) - Automated 5-stage development workflow

---
> Source: [Poorgramer-Zack/copilot-mcp-tool](https://github.com/Poorgramer-Zack/copilot-mcp-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
