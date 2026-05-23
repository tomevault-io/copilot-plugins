## branchhelios-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Helios** ("赫利俄斯") is a philosophical game project that creates a "consciousness exploration sandbox" - a "Prism of Consciousness" universe. Players invest their pure consciousness through unique belief systems, creating highly subjective experiences that together form an evolving shared reality.

The MVP goal is "Prism Heart" - a minimal world with 2 core NPCs and 1 simple scenario to validate the core experience loop: belief-based actions → cognitive dissonance → "Chamber of Echoes" self-reflection → belief evolution.

## Technical Architecture

- **Platform**: Vercel (unified deployment)
- **Frontend**: Next.js (`packages/web`)
- **Backend**: Python FastAPI on Vercel Serverless Functions (`packages/api`)
- **Database**: Supabase (PostgreSQL + pgvector)
- **Memory Engine**: Zep (conversation history)
- **AI Gateway**: Vercel AI Gateway (mandatory for all LLM calls)
- **Workflow Engine**: n8n (cognitive dissonance triggers)

## Monorepo Structure

```
packages/
├── web/          # Next.js frontend
└── api/          # Python/FastAPI backend
```

## Development Commands

### Setup and Installation
```bash
# Initial setup (from root)
npm install
```

### Local Development (Zero-Trust Mode)
```bash
# Frontend development (UI debugging)
npm run dev:web

# Backend development (API logic, requires: pip install uvicorn fastapi)
npm run dev:api
```

**Important**: Local development runs without API keys. External API calls will fail (expected behavior). Complete functionality testing is done via GitHub PR Vercel preview environments.

## Core System Components

### 1. Belief System (信念系统)
- **Belief DSL**: YAML/JSON format for defining character belief networks
- **Belief Compiler**: Python script converting belief files to LLM system prompts
- **Components**: World-view beliefs, Self-perception beliefs, Value beliefs
- **Evolution Tracking**: Records cognitive dissonance events in `agent_logs`

### 2. Agent Core (代理核心)
- **Primary API**: `/api/chat` (receives `player_id` and `message`)
- **Process Flow**: Load beliefs from Supabase → Retrieve conversation history from Zep → Call LLM via Vercel AI Gateway → Log interaction
- **Response Requirement**: Fast, belief-consistent NPC responses

### 3. Chamber of Echoes (回响之室)
- **API Endpoint**: `/api/echo` (receives `player_id` and `event_id`)
- **Function**: Generates subjective, first-person causal explanations based on player's belief system
- **Output**: Subjective attribution + 1-2 supporting "memory evidence" events

### 4. Director Engine (导演引擎)
- **Implementation**: n8n workflow monitoring `agent_logs`
- **Trigger Logic**: Detects cognitive dissonance (positive actions → negative feedback)
- **Action**: Inserts records into `events` table to trigger Chamber of Echoes opportunities

## Mandatory Development Contracts

### Environment Variables (Managed by Mike via Vercel)
**Backend Variables** (Python `os.environ.get()`):
- `VERCEL_AI_GATEWAY_URL`: AI Gateway endpoint
- `VERCEL_AI_GATEWAY_API_KEY`: Gateway authentication
- `SUPABASE_URL`: Database URL
- `SUPABASE_SERVICE_KEY`: Database service key
- `ZEP_API_KEY`: Memory service key

**Frontend Variables** (Next.js `process.env.`):
- `NEXT_PUBLIC_SUPABASE_URL`: Public database URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Public database key

**Note**: Never hardcode secrets. Variables are only available in Vercel preview/production environments, not locally.

### LLM Call Standard
All AI model calls **MUST** go through Vercel AI Gateway:

```python
import os
import requests

VERCEL_AI_GATEWAY_URL = os.environ.get("VERCEL_AI_GATEWAY_URL")
VERCEL_AI_GATEWAY_API_KEY = os.environ.get("VERCEL_AI_GATEWAY_API_KEY")

def call_llm(model_name: str, system_prompt: str, user_prompt: str):
    headers = {
        "Authorization": f"Bearer {VERCEL_AI_GATEWAY_API_KEY}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "model": model_name,
        "messages": [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ],
        "max_tokens": 2048
    }
    
    response = requests.post(f"{VERCEL_AI_GATEWAY_URL}/chat/completions", headers=headers, json=payload)
    response.raise_for_status()
    
    return response.json()["choices"][0]["message"]["content"]
```

## Git Workflow

### Branch Naming Convention
- `feature/[name]/[description]` - New features
- `fix/[name]/[description]` - Bug fixes
- Example: `feature/ethan/agent-core-base`

### Zero-Trust Development Workflow
1. **Sync**: `git checkout main && git pull origin main`
2. **Create branch**: `git checkout -b feature/your-name/your-feature`
3. **Local coding**: Use `npm run dev:web` or `npm run dev:api` for development
4. **Push**: `git push origin feature/your-name/your-feature`
5. **Create PR**: Submit PR with Vercel preview deployment link for cloud testing
6. **Code review**: Wait for Mike's approval and merge

**Critical**: 
- Never push directly to `main` branch
- Local development is for coding only
- Full functionality testing happens in Vercel preview environments

## MVP Scope

### ✅ In Scope
- Text-only frontend interface
- Simple belief system generation
- One scenario (tavern setting)
- Two NPCs with Belief DSL-defined personalities
- Complete core loop: conversation → logging → conflict detection → Chamber of Echoes trigger → subjective attribution

### ❌ Out of Scope
- Graphics, images, visual content
- Offline AI agents
- Complex world-building systems
- Multiple scenes/NPCs
- Combat, inventory, quest systems

## Success Criteria

1. **Belief Consistency**: NPCs demonstrate clear alignment between behavior and defined belief systems
2. **"Aha!" Moments**: Players experience "my thoughts created this outcome" realizations in Chamber of Echoes
3. **Technical Viability**: Full stack integration (Vercel, Supabase, n8n, Zep) operates smoothly

## Key Files to Watch

- `vercel.json`: Deployment configuration
- `packages/api/main.py`: FastAPI backend entry point
- `packages/api/requirements.txt`: Python dependencies
- `packages/web/`: Next.js frontend application

---
> Source: [xuelun999-hue/branchhelios-game](https://github.com/xuelun999-hue/branchhelios-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
