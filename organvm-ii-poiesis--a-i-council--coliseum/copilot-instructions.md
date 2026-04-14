## a-i-council-coliseum

> Welcome to the AI Council Coliseum project. This document provides the essential context, architectural patterns, and development workflows used in this codebase.

# AI Council Coliseum - Project Context

Welcome to the AI Council Coliseum project. This document provides the essential context, architectural patterns, and development workflows used in this codebase.

## Project Overview
AI Council Coliseum is a decentralized platform where AI agents debate real-time events. Viewers participate through blockchain-based voting and gamification.

- **Status:** MVP (In-memory backend, Next.js frontend scaffold)
- **Primary Tech Stack:**
  - **Backend:** Python 3.11, FastAPI, Uvicorn, Pytest
  - **Frontend:** Next.js 15, React 18, Tailwind CSS, pnpm, TypeScript, Zustand, Framer Motion
  - **Infrastructure:** Docker Compose
  - **Blockchain:** Solana & Ethereum (Planned integrations, current writes are `501`)

## Architecture
The system is designed with a modular architecture split into several core layers:

1. **API Gateway (FastAPI):** Handles REST and WebSocket endpoints.
2. **Service Layer:**
   - **AI Agent Framework:** 7 modules (Base Agent, Communication, Decision Engine, NLP, Knowledge Base, Memory Manager, Coordination).
   - **Event Pipeline:** 7 components (Ingestion, Classification, Prioritization, Routing, Processing, Storage, Notification).
   - **Voting System:** Voting engine, achievements, gamification, leaderboard.
3. **Data Layer:** Currently in-memory. Roadmap includes PostgreSQL and Redis.

### Core Component: System Orchestrator
The `SystemOrchestrator` (`backend/ai_agents/orchestrator.py`) is the central coordinator that manages agent lifecycles, event ingestion, and voting sessions.

## Building and Running

### Backend
Requires Python 3.11+.
```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r backend/requirements-test.txt
# runtime-only installs (Docker/production-style): pip install -r backend/requirements.txt
# Run the app
uvicorn backend.main:app --reload
```
- API Docs: `http://localhost:8000/docs`

### Frontend
Requires Node.js and pnpm.
```bash
cd frontend
pnpm install --frozen-lockfile
pnpm run dev
```
- App URL: `http://localhost:3000`

### Docker Compose
```bash
docker compose up -d
```
- Backend: `http://localhost:8000`
- Frontend: `http://localhost:3000`

## Development Conventions

- **Python Style:** Follow standard FastAPI/Pydantic patterns. Use async for I/O bound tasks.
- **Frontend Style:** Next.js App Router, functional components, Tailwind CSS for styling.
- **Testing:** 
  - Backend: `pytest` located in `backend/tests/`.
  - Frontend: `lint` and `build` checks.
- **Git Workflow:** Keep changes atomic. Do not commit secrets.

## MVP Capability Matrix
| Area | Status | Note |
|---|---|---|
| Agents | Working | Create/list/get/delete, in-memory memory snapshots |
| Events | Working | Ingest, list, in-memory classification/processing |
| Voting | Working | Session management, weighted voting, results |
| Gamification | Working | Achievements, leaderboards, user stats |
| Blockchain Writes | Unavailable | Returns `501` (Not Implemented) |

## Key Directories
- `backend/ai_agents/`: Core agent logic and orchestrator.
- `backend/api/`: FastAPI route definitions.
- `backend/event_pipeline/`: Event ingestion and processing components.
- `backend/voting/`: Voting engine and gamification logic.
- `frontend/src/app/`: Next.js application structure.
- `docs/`: Architectural documentation and ADRs.

## Roadmap & TODOs
- [ ] Replace in-memory state with durable persistence (PostgreSQL/Redis).
- [ ] Implement robust auth/authz for voting and user management.
- [ ] Integrate real blockchain write paths (Solana/Ethereum).
- [ ] Implement full frontend UI components (Agent views, Chat stream, Voting panel).
- [ ] Add WebSocket real-time updates for live streams.

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** ORGAN-II (Art) | **Tier:** standard | **Status:** PUBLIC_PROCESS
**Org:** `organvm-ii-poiesis` | **Repo:** `a-i-council--coliseum`

### Edges
- **Consumes** ← `organvm-ii-poiesis/performance-sdk`: dependency
- **Consumes** ← `organvm-ii-poiesis/metasystem-master`: dependency

### Siblings in Art
`core-engine`, `performance-sdk`, `example-generative-music`, `metasystem-master`, `example-choreographic-interface`, `showcase-portfolio`, `archive-past-works`, `case-studies-methodology`, `learning-resources`, `example-generative-visual`, `example-interactive-installation`, `example-ai-collaboration`, `docs`, `a-mavs-olevm`, `.github` ... and 14 more

### Governance
- Consumes Theory (I) concepts, produces artifacts for Commerce (III).

*Last synced: 2026-03-08T20:11:34Z*

## Session Review Protocol

At the end of each session that produces or modifies files:
1. Run `organvm session review --latest` to get a session summary
2. Check for unimplemented plans: `organvm session plans --project .`
3. Export significant sessions: `organvm session export <id> --slug <slug>`
4. Run `organvm prompts distill --dry-run` to detect uncovered operational patterns

Transcripts are on-demand (never committed):
- `organvm session transcript <id>` — conversation summary
- `organvm session transcript <id> --unabridged` — full audit trail
- `organvm session prompts <id>` — human prompts only


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | prompting-standards | Prompting Standards |
| system | any | research-standards-bibliography | APPENDIX: Research Standards Bibliography |
| system | any | research-standards | METADOC: Architectural Typology & Research Standards |
| system | any | sop-ecosystem | METADOC: SOP Ecosystem — Taxonomy, Inventory & Coverage |
| system | any | autopoietic-systems-diagnostics | SOP: Autopoietic Systems Diagnostics (The Mirror of Eternity) |
| system | any | cicd-resilience-and-recovery | SOP: CI/CD Pipeline Resilience & Recovery |
| system | any | cross-agent-handoff | SOP: Cross-Agent Session Handoff |
| system | any | document-audit-feature-extraction | SOP: Document Audit & Feature Extraction |
| system | any | essay-publishing-and-distribution | SOP: Essay Publishing & Distribution |
| system | any | market-gap-analysis | SOP: Full-Breath Market-Gap Analysis & Defensive Parrying |
| system | any | pitch-deck-rollout | SOP: Pitch Deck Generation & Rollout |
| system | any | promotion-and-state-transitions | SOP: Promotion & State Transitions |
| system | any | repo-onboarding-and-habitat-creation | SOP: Repo Onboarding & Habitat Creation |
| system | any | research-to-implementation-pipeline | SOP: Research-to-Implementation Pipeline (The Gold Path) |
| system | any | security-and-accessibility-audit | SOP: Security & Accessibility Audit |
| system | any | session-self-critique | session-self-critique |
| system | any | source-evaluation-and-bibliography | SOP: Source Evaluation & Annotated Bibliography (The Refinery) |
| system | any | stranger-test-protocol | SOP: Stranger Test Protocol |
| system | any | strategic-foresight-and-futures | SOP: Strategic Foresight & Futures (The Telescope) |
| system | any | typological-hermeneutic-analysis | SOP: Typological & Hermeneutic Analysis (The Archaeology) |

Linked skills: evaluation-to-growth


**Prompting (Google)**: context 1M tokens (Gemini 1.5 Pro), format: markdown, thinking: thinking mode (thinkingConfig)

<!-- ORGANVM:AUTO:END -->


## ⚡ Conductor OS Integration
This repository is a managed component of the ORGANVM meta-workspace.
- **Orchestration:** Use `conductor patch` for system status and work queue.
- **Lifecycle:** Follow the `FRAME -> SHAPE -> BUILD -> PROVE` workflow.
- **Governance:** Promotions are managed via `conductor wip promote`.
- **Intelligence:** Conductor MCP tools are available for routing and mission synthesis.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/organvm-ii-poiesis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/organvm-ii-poiesis)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
