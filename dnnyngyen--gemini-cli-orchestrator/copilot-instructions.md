## gemini-cli-orchestrator

> This is the gemini-cli-orchestrator project - a pure metaprompting MCP server that guides AI agents on how to collaborate with Google's Gemini AI model. It provides NO execution logic, only intelligent guidance.

# Gemini CLI Orchestrator - Development Context

## Project Overview
This is the gemini-cli-orchestrator project - a pure metaprompting MCP server that guides AI agents on how to collaborate with Google's Gemini AI model. It provides NO execution logic, only intelligent guidance.

## Core Philosophy
**Metaprompting-First**: Don't build intelligence into the system. Build prompts that elicit intelligence from the agent.

## Key Components
- **gemini-collaboration-guide.mjs**: Pure metaprompting MCP server that teaches collaboration strategies
- **Pure Guidance Tools**: Tools that return metaprompts, not execution results
- **Universal Portability**: Works anywhere the agent has bash access and gemini CLI installed

## Architecture
- **Teacher Model**: Provides strategic guidance rather than executing commands
- **No State Management**: Stateless guidance system with no workflow storage
- **No File Processing**: Agents handle file operations using their own tools
- **No Shell Execution**: Agents run gemini commands using their own bash capabilities

## Commands
```bash
node gemini-collaboration-guide.mjs    # Start pure metaprompting MCP server
npm run mcp                           # Start server
npm test                              # Run tests
```

## How It Works
1. Agent requests guidance using MCP tools
2. Server returns rich metaprompts that teach strategic thinking
3. Agent applies guidance to execute their own bash commands
4. Agent collaborates with gemini CLI directly using server's teaching

## Requirements
- External `gemini` CLI must be installed and configured on the agent's system
- Agent must have bash/shell access for executing gemini commands
- No server-side authentication or file access required

---
> Source: [dnnyngyen/gemini-cli-orchestrator](https://github.com/dnnyngyen/gemini-cli-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
