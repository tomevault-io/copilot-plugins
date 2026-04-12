## vibecheck

> **VibeCheck** is a learning-verification companion for AI-assisted workflows (vibecoding, AI-assisted design, AI-assisted writing, etc.). The core problem it solves: when you let AI do the heavy lifting, you often don't actually understand what was built. VibeCheck watches or ingests your AI session, then quizzes you to prove you do.

# VibeCheck — CLAUDE.md

## Project Identity

**VibeCheck** is a learning-verification companion for AI-assisted workflows (vibecoding, AI-assisted design, AI-assisted writing, etc.). The core problem it solves: when you let AI do the heavy lifting, you often don't actually understand what was built. VibeCheck watches or ingests your AI session, then quizzes you to prove you do.

---

## Autonomous Operation

Claude makes all implementation decisions in this project unless the user specifies otherwise. No confirmation needed for:
- Library/framework choices within the established stack
- File structure and naming conventions
- UI layout and component design
- Database schema decisions
- API design and endpoint structure
- Refactoring and code organization

Always ask the user before:
- Changing the core stack (Python backend, TSX frontend)
- Integrating paid third-party services beyond what's already in use
- Deleting files that contain user-created content or configuration

---

## Subagent Usage

For complex or parallelizable work, Claude should delegate to subagents via the Task tool. Use subagents for:

- **Codebase exploration** — Use the `Explore` subagent to search for files, patterns, or understand how a subsystem works before making changes
- **Implementation planning** — Use the `Plan` subagent when a feature touches 3+ files or requires architectural decisions
- **Parallel implementation** — When building a feature that has independent backend and frontend components, run both as parallel subagents simultaneously
- **Research tasks** — Delegate web searches or documentation lookups to a `general-purpose` subagent to keep the main context clean

Specifically, always use subagents for:
- Any new feature that spans both `backend/` and `frontend/` (run both in parallel)
- Deep codebase searches when you are not confident you know exactly where to look
- AI service calls for quiz/insight/analytics generation — prompt design is isolated in `services/`

---

## Feature Testing Requirements

After implementing any new feature or non-trivial change, Claude must verify it works before considering the task done:

1. **Backend features** — Start the backend server and make real HTTP requests to the new endpoint(s) using `curl`. Confirm expected status codes and response shapes.
2. **Frontend features** — If the frontend is running, confirm the relevant page/component renders without console errors. If not running, at minimum verify `npm run build` passes with no TypeScript errors.
3. **AI-integrated features** — Run an end-to-end smoke test: create a real session, trigger the AI call, and assert the output matches the expected JSON schema.
4. **Database changes** — After any schema change, verify migrations apply cleanly and at least one read + write round-trip succeeds.

If a test fails, fix it before moving on — do not leave the codebase in a broken state.

---

## Stack (non-negotiable)

| Layer     | Technology                                          |
|-----------|-----------------------------------------------------|
| Backend   | Python 3.12+, FastAPI, Uvicorn                      |
| Frontend  | React 18+, TypeScript, Vite, TailwindCSS            |
| Database  | SQLite via SQLAlchemy (async with aiosqlite)        |
| AI        | OpenAI SDK (gpt-4o) via OPENAI_API_KEY — see note  |
| Packaging | uv (Python), npm (frontend)                         |

**AI provider note:** The project currently uses OpenAI (gpt-4o). All AI logic is isolated in `backend/services/claude_service.py`. Switching to Anthropic Claude requires only: updating the env var, swapping the pip package, and rewriting that one file. See README for exact steps.

---

## Project Structure

```
vibecheck/
├── CLAUDE.md
├── README.md
├── VIBECHECK.md             # Architecture decisions + gotchas (living doc)
├── .env.example
├── hooks/
│   └── session_end.py       # Claude Code Stop hook (auto-captures + runs health)
├── backend/
│   ├── main.py              # FastAPI app entrypoint
│   ├── db.py                # DB init and session dependency
│   ├── mcp_server.py        # MCP server — 13 tools for Claude Code
│   ├── pyproject.toml
│   ├── models/
│   │   └── session.py       # Session, Quiz, Attempt, Insight, FocusArea, RepoGroup, Repo, RepoConnection, SessionHealth, SessionHandoff
│   ├── schemas/
│   │   └── session.py       # All Pydantic request/response schemas
│   ├── routers/
│   │   ├── sessions.py      # CRUD
│   │   ├── quiz.py          # Quiz generation
│   │   ├── results.py       # Attempt submission + results
│   │   ├── insights.py      # Session intelligence extraction
│   │   ├── analytics.py     # Cross-session analytics + catch-up
│   │   ├── codebase.py      # Directory scan + code-first quiz + self-brief
│   │   ├── health.py        # Context rot analysis (POST generate / GET fetch)
│   │   ├── handoff.py       # Handoff doc (POST generate / GET fetch / POST apply)
│   │   └── multi_repo.py    # Repo group CRUD + cross-repo analysis
│   └── services/
│       ├── claude_service.py    # All AI provider calls (quiz, eval, insights, health, handoff, brief)
│       ├── quiz_engine.py       # Orchestrates DB + AI for quizzes
│       ├── insights_service.py  # Session intelligence extraction
│       ├── analytics_service.py # Cross-session topic scoring + catch-up
│       ├── codebase_service.py  # Directory scanning + code-first quiz gen
│       ├── self_brief_service.py    # AI codebase onboarding brief
│       ├── context_rot_service.py   # Context health analysis + persistence
│       ├── handoff_service.py       # Handoff doc generation + persistence
│       └── multi_repo_service.py    # Cross-repo connection analysis
└── frontend/
    ├── src/
    │   ├── api/sessions.ts  # All typed API fetch wrappers
    │   └── pages/
    │       ├── Home.tsx
    │       ├── NewSession.tsx
    │       ├── SessionDetail.tsx
    │       ├── Quiz.tsx
    │       ├── Results.tsx
    │       ├── Insights.tsx         # Session intelligence + CLAUDE.md apply
    │       ├── Analytics.tsx        # Comprehension dashboard + catch-up briefs
    │       ├── CodebaseMap.tsx      # Codebase scanner + focus areas
    │       ├── SelfBrief.tsx        # AI codebase brief + CLAUDE.md apply
    │       ├── SelfBriefComponents.tsx
    │       ├── MultiRepo.tsx        # Multi-repo group manager
    │       ├── MultiRepoComponents.tsx
    │       └── SessionHealth.tsx    # Context rot report + handoff generation
    ├── index.html
    ├── vite.config.ts
    └── package.json
```

---

## Data Models

| Model | Key fields |
|---|---|
| `Session` | title, transcript, source_type (claude_code/chatgpt/cursor/generic/code_file), status |
| `Quiz` | session_id, questions (JSON) |
| `Attempt` | session_id, quiz_id, answers (JSON), evaluations (JSON), score, feedback_summary |
| `Insight` | session_id, decisions/patterns/gotchas/proposed_rules (all JSON) |
| `FocusArea` | type (file/concept), value, label |
| `SessionHealth` | session_id, efficiency_score, lazy_prompt_count, estimated_wasted_token_ratio, lazy_prompts (JSON), breakpoints (JSON), summary |
| `SessionHandoff` | session_id, content (text), word_count |

---

## API Design

```
# Sessions
POST   /api/sessions
GET    /api/sessions
GET    /api/sessions/{id}
DELETE /api/sessions/{id}

# Quiz
POST   /api/sessions/{id}/quiz       # Generate quiz (calls AI)
GET    /api/sessions/{id}/quiz

# Attempts & Results
POST   /api/sessions/{id}/attempt    # Submit answers (AI evaluates)
GET    /api/sessions/{id}/results

# Session Intelligence
POST   /api/sessions/{id}/insights   # Extract decisions/patterns/gotchas (AI)
GET    /api/sessions/{id}/insights
POST   /api/sessions/{id}/insights/apply  # Write to a CLAUDE.md file

# Context Health
POST   /api/sessions/{id}/health     # Analyze for context rot (AI) — 409 if exists
GET    /api/sessions/{id}/health     # Fetch cached health report

# Session Handoff
POST   /api/sessions/{id}/handoff    # Generate compact handoff doc (AI) — 409 if exists
GET    /api/sessions/{id}/handoff    # Fetch cached handoff doc
POST   /api/sessions/{id}/handoff/apply  # Write handoff to a file (overwrites)

# Analytics
GET    /api/analytics                # Cross-session topic scores + blind spots
POST   /api/analytics/catchup        # Generate personalized catch-up brief (AI)

# Codebase
POST   /api/codebase/scan            # Scan directory for comprehension risk (AI)
POST   /api/codebase/quiz            # Generate quiz from a code file (AI)
POST   /api/codebase/brief           # AI onboarding brief for a directory
POST   /api/codebase/brief/apply     # Append brief to a CLAUDE.md file

# Focus Areas
GET    /api/focus
POST   /api/focus
DELETE /api/focus/{id}

# Multi-Repo
GET    /api/repos/groups
POST   /api/repos/groups
GET    /api/repos/groups/{id}
POST   /api/repos/groups/{id}/analyze
GET    /api/repos/groups/{id}/context
```

---

## Frontend Pages

1. **Home** (`/`) — Session list + nav to Analytics and Codebase Map
2. **New Session** (`/sessions/new`) — Paste transcript, select source type
3. **Session Detail** (`/sessions/:id`) — Transcript, quiz actions, insights trigger, context health button
4. **Quiz** (`/sessions/:id/quiz`) — Step-through question interface
5. **Results** (`/sessions/:id/results`) — Score, per-answer feedback, retry
6. **Insights** (`/sessions/:id/insights`) — Decisions, patterns, gotchas, CLAUDE.md apply
7. **Session Health** (`/sessions/:id/health`) — Context rot report + handoff generation
8. **Analytics** (`/analytics`) — Topic scores, blind spots, catch-up briefs
9. **Codebase Map** (`/codebase`) — Directory scanner, focus areas, code-first quiz
10. **Self-Brief** (`/self-brief`) — AI codebase onboarding brief + CLAUDE.md apply
11. **Multi-Repo** (`/multi-repo`) — Repo group manager + cross-repo connection analysis

---

## MCP Tools (Claude Code integration)

| Tool | Description |
|---|---|
| `vibecheck_capture` | Create session from transcript + generate quiz |
| `vibecheck_sessions` | List recent sessions |
| `vibecheck_results` | Get score + feedback for a session |
| `vibecheck_insights` | Extract session intelligence |
| `vibecheck_blind_spots` | Cross-session analytics + blind spot list |
| `vibecheck_catchup` | Personalized catch-up brief for a weak topic |
| `vibecheck_scan` | Scan directory for comprehension risk |
| `vibecheck_code_quiz` | Generate quiz from a code file |
| `vibecheck_self_brief` | AI codebase onboarding brief with sub-agent suggestions |
| `vibecheck_apply_brief` | Append self-brief to a CLAUDE.md file |
| `vibecheck_repo_context` | Cross-repo connection context for a named repo group |
| `vibecheck_health` | Analyze session for context rot — efficiency score, lazy prompts, breakpoints |
| `vibecheck_handoff` | Generate <500-word handoff doc for fresh-session continuity |

---

## Code Style

### Python
- Type hints everywhere, no bare `dict` return types
- Pydantic v2 models for all request/response shapes (`ConfigDict(from_attributes=True)`)
- Async all the way down (async def routes, async SQLAlchemy)
- Services are plain async functions, not classes
- No print statements — use Python `logging`
- AI clients use lazy singleton `_get_client()` pattern

### TypeScript/React
- Functional components only, no class components
- `const` arrow functions for components
- TailwindCSS utility classes only — no custom CSS files
- API calls live in `src/api/sessions.ts` — never fetch directly from components
- No `any` types — write proper interfaces

### General
- No TODO comments in committed code — either do it or create a task
- Keep files under 300 lines — split when approaching limit
- Prefer explicit over clever

---

## Environment Variables

```
# AI provider — choose one:
OPENAI_API_KEY=sk-proj-...          # Current default (OpenAI gpt-4o)
# ANTHROPIC_API_KEY=sk-ant-...      # Alternative — see README to switch

DATABASE_URL=sqlite+aiosqlite:///./vibecheck.db
CORS_ORIGINS=http://localhost:5173
```

---

## Running the Project

```bash
# Backend
cd backend && uv run uvicorn main:app --reload

# Frontend
cd frontend && npm run dev
```

Backend: `http://localhost:8000` · Frontend: `http://localhost:5173`

---

## What This Is NOT

- Not a plagiarism detector
- Not a code quality tool
- Not an AI tutor that teaches — it only verifies understanding
- Not a replacement for actually reading what the AI wrote

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/githipo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/githipo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
