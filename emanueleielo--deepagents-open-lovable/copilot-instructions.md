## deepagents-open-lovable

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a deepagents-based frontend development platform. It consists of two main parts:

1. **`agent/`** - Python backend: LangGraph-based AI agent for frontend development
2. **`gui/`** - React/Vite frontend: Custom UI for interacting with the agent

## Common Commands

### Agent (Python Backend)

```bash
# Start the LangGraph development server (from agent/)
cd agent && langgraph dev

# The agent runs at http://localhost:2024
# LangGraph Studio UI at https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

Environment variables (in `agent/.env`):
- `ANTHROPIC_API_KEY` - Required for Claude models
- `MODEL` - Model to use (default: `anthropic:claude-sonnet-4-5-20250929`)
- `TAVILY_API_KEY` - Optional, for web search

### GUI (Vite/React Frontend)

```bash
# From gui/
npm install
npm run dev      # Dev server at http://localhost:5173
npm run build    # Build for production
npm run lint     # Run ESLint
```

Environment variables (in `gui/.env`):
- `VITE_API_URL` - LangGraph API URL (default: http://localhost:2024)
- `VERCEL_API_TOKEN` - Optional, for preview deployments

## Architecture

### Agent Architecture (`agent/src/`)

The agent is built on the `deepagents` library which provides:
- **Built-in tools**: `write_todos`, `read_todos`, `ls`, `read_file`, `write_file`, `edit_file`, `glob`, `grep`, `task`
- **Built-in middleware**: TodoListMiddleware, FilesystemMiddleware, SubAgentMiddleware, SummarizationMiddleware

Key files:
- `agent.py` - Creates the frontend agent using `create_deep_agent()` with StateBackend
- `prompts.py` - System prompt with React/Next.js/Tailwind/shadcn stack guidance
- `subagents.py` - Defines `designer` and `image-researcher` sub-agents
- `tools.py` - Custom tools: `http_request`, `web_search`, `fetch_url`
- `skills/` - Skill YAML/MD files loaded by SkillsMiddleware

Backend routes (all StateBackend - stored in LangGraph state):
- `/app/` - App/code files (shown in filesystem panel)
- `/memory/` - Agent memory files
- Default falls back to `/app/`

### GUI Architecture (`gui/src/`)

Vite + React 18 + TypeScript + Tailwind CSS + react-router-dom

- `pages/` - Route components (`NewThread`, `ThreadView`)
- `components/` - UI components organized by feature (chat, filesystem, preview, layout)
- `hooks/` - Custom hooks (`useChat`, `useVercelDeploy`, `useDebouncedValue`)
- `api/` - API client for LangGraph backend
- `providers/` - React context providers

Vite proxies `/api` to `http://localhost:2024` (LangGraph backend).

### Deepagents Patterns

When extending the agent:
- Add custom tools in `agent/src/tools.py` using `@tool` decorator
- Add sub-agents in `agent/src/subagents.py` as `SubAgent` dicts
- Add skills as SKILL.md files in `agent/skills/<skill-name>/`
- Middleware injects tool instructions automatically - don't duplicate in system prompts

The agent uses `CompositeBackend` with `StateBackend` so all files are stored in LangGraph state and streamed to the UI (no persistent filesystem).

## Key Technologies

**Agent**: Python 3.11+, deepagents, LangChain, LangGraph, langchain-anthropic, tavily-python

**GUI**: React 18, TypeScript, Vite, Tailwind CSS, @langchain/langgraph-sdk, react-router-dom, Sandpack

---
> Source: [emanueleielo/deepagents-open-lovable](https://github.com/emanueleielo/deepagents-open-lovable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
