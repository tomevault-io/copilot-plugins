## directive

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Executive Order** is a multi-agent AI collaboration framework based on the US Constitutional system (1787). It implements a 10-agent architecture with separation of powers, checks and balances, and judicial review - features missing from other frameworks like CrewAI, AutoGen, and Edict.

**Core Innovation:** Supreme Court agent for dispute arbitration - the first framework with built-in judicial review for inter-agent conflicts.

## Architecture

### 10-Agent Flow
```
President → Chief of Staff (triage) → NSC (planning) → Senate (review/filibuster)
→ OMB (dispatch) → 5 Departments (parallel execution) → Supreme Court (disputes)
```

### Agent IDs and Roles
- `chief_of_staff` - Message triage, task identification
- `nsc` - Strategic planning, task decomposition
- `senate` - Quality gate, approve or filibuster plans
- `omb` - Task dispatch, progress tracking, result consolidation
- `treasury` - Data analysis, reporting, cost accounting
- `state_dept` - Documentation, technical writing, communication
- `dod` - Engineering implementation, code development
- `doj` - Security compliance, audit scanning
- `commerce` - CI/CD, infrastructure, deployment
- `supreme_court` - Dispute resolution, final arbitration

### Key Directories
- `agents/*/SOUL.md` - Agent system prompts and behavioral definitions
- `dashboard/` - React dashboard (built) + Python API server
- `directive/frontend/` - React 18 + Vite + TypeScript dashboard source
- `directive/backend/` - Backend launcher (reuses dashboard/server.py)
- `scripts/` - Python utilities for data sync, model config, skills management
- `data/` - Runtime JSON data (tasks, agent config, stats)

## Development Commands

### Start Backend
```bash
# From project root
python3 dashboard/server.py --port 7891

# Or via directive launcher
python3 directive/backend/server.py --port 7891
```

### Start Frontend (Development)
```bash
cd directive/frontend
npm install
npm run dev
# Opens on http://localhost:5173, proxies to backend at :7891
```

### Build Frontend
```bash
cd directive/frontend
npm run build
# Output: directive/frontend/dist/
```

### Run Tests
```bash
# Python tests
python3 -m pytest tests/

# Specific test
python3 -m pytest tests/test_e2e_kanban.py -v
```

### Data Refresh Loop
```bash
# Continuous 15s polling for live dashboard updates
bash scripts/run_loop.sh
```

## Key Technical Details

### Backend API (dashboard/server.py)
- Port: 7891 (configurable via `--port`)
- Zero external dependencies (Python stdlib only)
- File-based JSON storage with atomic writes via `scripts/file_lock.py`
- CORS enabled for localhost:5173 (Vite dev) and localhost:7891

**Main Endpoints:**
- `GET /api/live-status` - Task states, sync status
- `GET /api/agent-config` - Agent metadata, model assignments
- `POST /api/set-model` - Hot-swap agent LLM models
- `POST /api/task-action` - Pause/resume/cancel tasks
- `GET /api/officials-stats` - Agent performance metrics

### Frontend Stack
- React 18.3 + TypeScript 5.6
- Vite 6.0 for dev/build
- Zustand for state management
- Tailwind CSS 3.4 for styling
- Lucide React for icons
- Auto-polling every 5s via `store.ts`

### Data Files (data/)
- `tasks_source.json` - Task definitions and state
- `agent_config.json` - Agent metadata, model assignments
- `live_status.json` - Computed dashboard state
- `officials_stats.json` - Agent performance metrics
- `model_change_log.json` - Model swap history

### Agent Configuration
Agents are defined in `data/agent_config.json` with:
- `id` - Unique agent identifier
- `model` - Current LLM model (e.g., "anthropic/claude-sonnet-4-6")
- `allowAgents` - Communication permissions (constitutional constraints)
- `workspace` - OpenClaw workspace path
- `skills` - Installed skill packages

## Important Patterns

### Task State Machine
```
pending → planning → under_review → [filibustered | dispatched]
→ in_progress → pending_review → [blocked | completed]
```

- `filibustered` - Senate rejected, NSC must revise
- `blocked` - Dispute escalated to Supreme Court
- `completed` - Archived as Executive Record

### File Locking
Always use atomic operations from `scripts/file_lock.py`:
```python
from file_lock import atomic_json_read, atomic_json_write, atomic_json_update

data = atomic_json_read(path, default={})
atomic_json_write(path, data)
atomic_json_update(path, lambda d: {**d, 'key': 'value'})
```

### Agent Communication Constraints
Agents can only message others in their `allowAgents` list. This enforces constitutional separation of powers:
- Chief of Staff → NSC only
- NSC → Senate only
- Senate → NSC (reject) or OMB (approve)
- OMB → 5 departments + Supreme Court
- Departments → OMB or Supreme Court (appeals)

## Common Tasks

### Add New Agent
1. Create `agents/{agent_id}/SOUL.md` with system prompt
2. Add agent entry to `data/agent_config.json`
3. Update `allowAgents` for relevant agents
4. Restart backend

### Change Agent Model
Via dashboard: Models tab → select agent → choose model → apply
Via API: `POST /api/set-model` with `{agentId, model}`

### Debug Task Flow
1. Check `data/tasks_source.json` for task state
2. Check `data/live_status.json` for computed status
3. Review agent SOUL.md files for behavioral rules
4. Check backend logs for API errors

### Update Dashboard UI
1. Edit files in `directive/frontend/src/`
2. Components are in `directive/frontend/src/components/`
3. State management in `directive/frontend/src/store.ts`
4. Run `npm run dev` for hot reload
5. Build with `npm run build` when ready

## Dependencies

### Backend
- Python 3.9+ (stdlib only, zero pip dependencies)

### Frontend
- Node.js 18+ / npm 9+
- Dependencies in `directive/frontend/package.json`

### Runtime
- OpenClaw framework (agents run in OpenClaw workspaces)
- LLM API keys configured in OpenClaw

## Project Philosophy

This framework treats AI agents as government officials with:
- **Defined jurisdictions** (agent roles)
- **Communication protocols** (allowAgents constraints)
- **Quality gates** (Senate review)
- **Dispute resolution** (Supreme Court)
- **Audit trails** (Congressional Record)

The 1787 US Constitution solved multi-party coordination without a central authority. This framework applies those same principles to multi-agent AI systems.

---
> Source: [sanbuphy/directive](https://github.com/sanbuphy/directive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
