## smartfin-v2

> This is an agentic workflow for personal financial management, built with LangGraph (supervisor pattern), including 1 supervisor and 5 specialized agents where each is responsible for a specific task.

# CLAUDE.md

## Project Overview
This is an agentic workflow for personal financial management, built with LangGraph (supervisor pattern), including 1 supervisor and 5 specialized agents where each is responsible for a specific task.

### Tech Stacks
- Anthropic Claude API - for LLM invocation
- React + Vite - for the chat interface
- FastAPI - for communications between the backend and frontend
- Docker Compose - for services containerizations
- PostgreSQL - for AppState checkpoint and session persistence
- Redis - for caching repetitive User input - LLM responses

---

## Key Commands

```bash
git pull --rebase
cp .env.example .env        # fill in ANTHROPIC_API_KEY, LANGSMITH_API_KEY

# create venv
uv venv

# activate venv
# for Windows Powershell
.venv/Scripts/activate.ps1 
# for Windows Bash
.venv/Scripts/activate
# for Linux/MacOS
source .venv/bin/activate

# install dependencies
uv pip install -r requirements.txt

# start backend & frontend services
docker compose up -d

# run UI
cd ui
npm install
npm run dev
```

Model selection is defined in `config/model_registry.json`; there is no separate
`SMARTFIN_MODEL` environment variable anymore.

### LiteLLM Gateway — Virtual Key Setup

The backend uses a scoped virtual key (`LITELLM_VIRTUAL_KEY`) instead of raw API keys.
On a fresh install the key must be created once after `docker compose up -d`:

```bash
# Issue a virtual key with access to all registered models
curl -X POST http://localhost:4000/key/generate \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "models": ["claude-haiku-4-5", "claude-opus-4-8", "claude-sonnet-4-6",
               "claude-sonnet-4-5", "text-embedding-3-small"],
    "key_alias": "smartfin-backend"
  }'
# Copy the returned "key" value into LITELLM_VIRTUAL_KEY in .env
```

When new models are added to `gateway/config.yaml`, update the existing key:

```bash
curl -X POST http://localhost:4000/key/update \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"key": "<LITELLM_VIRTUAL_KEY>", "models": ["claude-haiku-4-5", "claude-opus-4-8",
       "claude-sonnet-4-6", "claude-sonnet-4-5", "text-embedding-3-small"]}'
```

---

## Key References Index

| Topic | Path |
| --- | --- |
| Environment variables | `@.env.example` |
| Coding Principles | `@.claude/rules/code-principle.md` |
| System Design Practices | `@.claude/rules/system-design.md`|
| Agent responsibilities and implementation notes | `@.claude/rules/agents.md` |
| Orchestrator entry point and public API | `@.claude/rules/orchestrator.md` |
| Test commands and coverage | `@.claude/rules/tests.md` |

<!-- Optional documents:
| Git workflow, branch naming, commit format | `CONTRIBUTING.md` |
| UI structure and page descriptions | `ui/README.md` |
-->

---

## Key Caveats

<!--- Run `git pull --rebase` to update from the remote repository
 - Never commit or push directly to `main`
- All changes must be branched off `dev`
- Branch type and commit type must match → see `CONTRIBUTING.md`-->
- Ask me for further details as long as my request is ambiguous until you've collected all required information. Do this before any write operation.
- **NEVER READ** `@.env`
- **YOU MUST** activate the venv and run docker compose to start the backend and frontend services, and to run a test. This is important
- Follow `@.claude/rules/code-principle.md` during any code modification that include complex dependencies and multiple modules
- Follow `@.claude/rules/system-design.md` when designing or refactoring a service
- Use **Plan Mode** before modifying core modules. Split the task. Show me the plan details with reasoning **By Subtask**. DO NOT execute directly, even with auto-accept turned on
- Use Subagent-Driven Development. Specify each subtask to a subagent. Run subagents in parallel or sequence according to the dependencies between subtasks
- Always check the code syntax after every modification and do a validation yourself

---
> Source: [junkaijunkai/SmartFin-v2](https://github.com/junkaijunkai/SmartFin-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
