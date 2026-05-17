## stock-news-sentiment-analysis

> Welcome to the project! This file serves as the "Mission Control" for agentic development.

# Stock News Sentiment Analysis

Welcome to the project! This file serves as the "Mission Control" for agentic development.

## Project Overview
A 3-microservice application that visualizes stock price timelines overlaid with news sentiment data.
- **Frontend**: Angular 17, TradingView Lightweight Charts
- **Backend**: Go `stock-service` (net/http, go-cache, orchestrator)
- **ML Service**: Python `sentiment-service` (FastAPI, ProsusAI/finbert)

## Relevant Documents
- **Specification:** `docs/stock-news-sentiment-analysis-spec.md`
- **Implementation Plan:** `docs/there-is-a-stock-news-sentiment-analysis-precious-sutton.md`

## Agent Execution Model & Delegation
We use specialized subagents for development. If you are an agent planning work, dispatch the subagent that best matches the component:

| Task | Component | Agent to Dispatch (`.agent/agents/`) |
|------|-----------|-------------------------------------|
| 1 | `sentiment-service/` | `sentiment-analysis-engineer` |
| 2 | `stock-service/` | `go-backend-architect` |
| 3 | `frontend/` | `angular-frontend-dev` |
| 4 | `docker-compose.yml` | `cicd-docker-engineer` |
| All | Cross-service architecture | `microservice-architect` |

**Execution Order:**
- Tasks 1 & 2 can run in **parallel** (independent API contracts).
- Task 3 runs **after** Tasks 1 & 2 are complete.
- Task 4 runs **last** to wire everything together.

## Coordination Protocol
1. **Plan Adherence:** You MUST follow the **Implementation Plan** (`docs/there-is-a-stock-news-sentiment-analysis-precious-sutton.md`) meticulously.
2. **Track Progress:** Tick off the `- [ ]` checkboxes in the implementation plan as steps are completed.
3. **Persist Context:** Update your specific persona's `MEMORY.md` index file inside `.agent/agent-memory/` whenever you establish project context, architectural decisions, or preferences.
4. **Environment:** The host OS is Windows. The API key is stored in the local `.env` file (do not commit it).
5. **Testing:** Follow the test-driven development (TDD) approach outlined in the plan. Write tests first, ensure they fail, implement the code, and then pass the tests.

---
> Source: [LosEvons/stock-news-sentiment-analysis](https://github.com/LosEvons/stock-news-sentiment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
