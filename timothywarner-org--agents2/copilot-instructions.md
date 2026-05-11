## agents2

> - **Purpose:** O'Reilly Live Learning course (4 hours) demonstrating production-ready AI agent patterns for HR resume screening, using LangGraph (stateful orchestration) and CrewAI (multi-agent collaboration), fully coupled with Azure AI Foundry.

# Copilot Instructions for agents2 (O'Reilly AI Agents Training)

## Project Overview

- **Purpose:** O'Reilly Live Learning course (4 hours) demonstrating production-ready AI agent patterns for HR resume screening, using LangGraph (stateful orchestration) and CrewAI (multi-agent collaboration), fully coupled with Azure AI Foundry.
- **Primary Project:** `contoso-hr-agent/` -- Contoso HR Agent that screens Microsoft Certified Trainer resumes.
- **Legacy Reference:** `oreilly-agent-mvp/` -- Earlier iteration (GitHub issue triage pipeline); kept for reference only. Do not modify unless explicitly requested.

## Structure

- `contoso-hr-agent/` is the main Python package.
  - `src/contoso_hr/` contains all core logic:
    - `pipeline/` -- LangGraph + CrewAI orchestration (graph.py, agents.py, tasks.py, tools.py, prompts.py)
    - `knowledge/` -- ChromaDB vector store (vectorizer.py) and document retrieval (retriever.py)
    - `watcher/` -- Folder watcher for event-driven resume processing
    - `mcp_server/` -- FastMCP 2 server (SSE on port 8081)
    - `memory/` -- SQLite store (sqlite_store.py) and LangGraph checkpoints (checkpoints.py)
    - `util/` -- Port utilities
  - `web/` -- Three HTML pages: chat.html, candidates.html, runs.html
  - `sample_resumes/` -- Test resumes following `RESUME_*.txt` naming convention
  - `sample_knowledge/` -- Knowledge base documents (supports `.pdf`, `.docx`, `.pptx`, `.md`)
  - `data/` -- Runtime directory (chat_sessions/, incoming/, outgoing/, chroma/, hr.db, checkpoints.db)

## Key Workflows

- **Run the HR engine (FastAPI on port 8080):** `uv run hr-engine`
- **Run the folder watcher (event-driven resume processing):** `uv run hr-watcher`
- **Run the MCP server (FastMCP 2 on port 8081):** `uv run hr-mcp`
- **Seed the knowledge base into ChromaDB:** `uv run hr-seed`
- **Run tests:** `uv run pytest tests/ -v`
- **Lint/format:** `uv run ruff check src/ tests/` and `uv run ruff format src/ tests/`
- **Environment:** Copy `.env.example` to `.env` and configure Azure AI Foundry credentials
- **Dependencies:** Managed via `pyproject.toml`; use `uv` for all package management

## Tech Stack

- **Orchestration:** LangGraph StateGraph + CrewAI (fully coupled pipeline)
- **LLM:** Azure AI Foundry (gpt-4-1-mini via AzureChatOpenAI / LiteLLM)
- **Embeddings:** Azure AI Foundry (text-embedding-3-large via AzureOpenAIEmbeddings)
- **Vector Store:** ChromaDB (local)
- **Database:** SQLite (hr.db for evaluations, checkpoints.db for LangGraph state)
- **MCP Server:** FastMCP 2 (SSE transport)
- **API Framework:** FastAPI
- **Web Search:** Brave Search API
- **Package Manager:** uv

## Pipeline Architecture (Parallel)

The pipeline uses LangGraph fan-out/fan-in for parallel subagent execution:

```text
intake -> [policy_expert || resume_analyst] -> decision_maker -> notify
```

- `intake`: Validates ResumeSubmission
- `policy_expert` + `resume_analyst`: Run CONCURRENTLY (LangGraph fan-out)
- `decision_maker`: Receives merged results (LangGraph fan-in)
- `notify`: Assembles EvaluationResult, logs summary

## CrewAI Agents

| Agent | Role | Tools |
|-------|------|-------|
| ChatConciergeAgent ("Alex") | Interactive HR Q&A via /api/chat | query_hr_policy |
| PolicyExpertAgent | Evaluates resumes against HR policies (ChromaDB) | query_hr_policy |
| ResumeAnalystAgent | Researches candidate background and credentials | brave_web_search |
| DecisionMakerAgent | Synthesizes findings into hiring recommendation | None (pure reasoning) |

## Four Dispositions

Strong Match | Possible Match | Needs Review | Not Qualified

## Data Model Chain

ResumeSubmission -> PolicyContext -> CandidateEval -> HRDecision -> EvaluationResult

## Chat History

- **Frontend:** Stored in `localStorage` for instant restore
- **Backend:** Persisted as JSON files in `data/chat_sessions/{session_id}.json`
- **Context:** Last 20 turns included as transcript in each concierge task prompt

## Web Pages

- `chat.html` -- Chat with Alex + resume upload, session management, suggestion buttons
- `candidates.html` -- Results grid of evaluated candidates
- `runs.html` -- Pipeline Runs trace showing live parallel branch execution

## Patterns and Conventions

- **No business logic in `__init__.py`** -- only imports and metadata
- **Scripts registered in `pyproject.toml`** for CLI use via `uv run`
- **Secrets:** Never commit `.env` or secrets; use environment variables
- **Resumes:** All sample resumes use `RESUME_*.txt` naming in `sample_resumes/`
- **Knowledge base:** Place `.pdf`, `.docx`, `.pptx`, or `.md` files in `sample_knowledge/`
- **One Crew.kickoff() per LangGraph node** -- no nested orchestration
- **Pydantic v2** for all data models (`model_dump()`, `model_validate_json()`)
- **Line length:** 100 characters (configured in ruff)
- **`data/` directories are runtime-only** -- never commit their contents

## Integration Points

- **Azure AI Foundry:** LLM and embedding provider (all three client types share endpoint/key from .env)
- **ChromaDB:** Local vector store for policy/knowledge retrieval
- **Brave Search:** External web search for candidate research
- **FastMCP 2:** Exposes agent tools as MCP endpoints (SSE on port 8081)

## Examples

- To process a new resume: place a `RESUME_*.txt` file in `data/incoming/` and run `uv run hr-watcher`, or upload via the web UI
- To add a new agent: extend `pipeline/agents.py`, add a task in `pipeline/tasks.py`, wire into `pipeline/graph.py`
- To add knowledge: place documents in `sample_knowledge/` and run `uv run hr-seed`

## References

- See `contoso-hr-agent/pyproject.toml` for dependencies, scripts, and build config
- See `.env.example` for environment setup
- See `contoso-hr-agent/src/contoso_hr/` for all core logic
- See `CLAUDE.md` at the repo root for the full architecture reference

---

_Keep instructions concise and up-to-date. Update this file if workflows or architecture change._

---
> Source: [timothywarner-org/agents2](https://github.com/timothywarner-org/agents2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
