## presentation-app

> AI-powered PowerPoint generation agent. Chat interface to create and edit presentations.

# Presentation App

AI-powered PowerPoint generation agent. Chat interface to create and edit presentations.

## Stack

- **Backend**: FastAPI + Claude Agent SDK + python-pptx
- **Frontend**: Next.js + React + TypeScript + Tailwind CSS
- **Architecture**: Forked from [form-filling-exp](https://github.com/jerryjliu/form_filling_app)

## Key Files

```
backend/
  agent.py          # Claude Agent SDK, MCP tools, system prompts
  main.py           # FastAPI server, SSE streaming endpoints
  pptx_processor.py # python-pptx operations (create, edit, export)

web/src/
  app/page.tsx      # Main app component
  lib/api.ts        # API client with SSE streaming
  lib/session.ts    # Session persistence
```

## Commands

```bash
# Backend
cd backend && pip install -r requirements.txt
cd backend && python main.py  # runs on :8000

# Frontend
cd web && npm install
cd web && npm run dev         # runs on :3000
```

## Context Files

- `research.md` - Comprehensive architecture research, patterns from form-filler, competitive analysis
- `plans/*.md` - Implementation plans with phases and success criteria
- `research/*.md` - Research documents from `/research` command
- `.claude/history/*.md` - Compacted conversation summaries

## Custom Commands

- `/create_plan <task>` - Create detailed implementation plan
- `/implement_plan <path>` - Execute plan phase-by-phase
- `/research <topic>` - Research codebase, output to research/

---
> Source: [jerryjliu/presentation_app](https://github.com/jerryjliu/presentation_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
