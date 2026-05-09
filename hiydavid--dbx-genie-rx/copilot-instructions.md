## dbx-genie-rx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GenieRX is an LLM-powered linting tool that analyzes Databricks Genie Space configurations against best practices. It evaluates 10 configuration sections using LLM-based evaluation, provides severity-based findings with remediation guidance, and outputs compliance scores.

**Key design principle**: `docs/checklist-by-schema.md` is the single source of truth for all checklist items. Non-developers can customize checks by editing this markdown file.

## Development Commands

```bash
# Initial setup (creates .env.local, sets up MLflow experiment)
./scripts/quickstart.sh

# Backend server (localhost:8000, serves API + pre-built frontend)
uv run start-server

# Backend with hot-reload (localhost:5001)
uv run uvicorn agent_server.start_server:app --reload --port 5001

# Frontend development (localhost:5173, proxies to backend on 5001)
npm run dev

# Frontend linting and type-check
npm run lint
npm run build  # TypeScript checked during build

# Build frontend for production (outputs to dist/)
./scripts/build.sh

# Deploy to Databricks Apps
./scripts/deploy.sh genie-space-analyzer
```

## Architecture

```text
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────────┐
│  React Frontend │────▶│  FastAPI + Agent     │────▶│ Databricks LLM   │
│  (src/)         │     │  (agent_server/)     │     │ (Claude Sonnet)  │
└─────────────────┘     └──────────────────────┘     └──────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        ▼                        ▼                        ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Databricks API   │  │  Best Practices  │  │  MLflow Traces   │
│ (Genie Spaces)   │  │  (docs/*.md)     │  │  (Databricks)    │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### Core Components

| File | Purpose |
| ------ | --------- |
| `agent_server/agent.py` | `GenieSpaceAnalyzer` class with LLM integration, MLflow tracing, streaming |
| `agent_server/optimizer.py` | `GenieSpaceOptimizer` class for generating field-level optimization suggestions |
| `agent_server/api.py` | REST API endpoints for frontend (`/api/space/fetch`, `/api/analyze/section`, `/api/optimize`, `/api/config/merge`, etc.) |
| `agent_server/checklist_parser.py` | Parses `docs/checklist-by-schema.md` to extract checklist items by section |
| `agent_server/checks.py` | Resolves checklist items per section using checklist_parser |
| `agent_server/auth.py` | OBO authentication via `contextvars` for Databricks Apps, PAT/OAuth for local dev |
| `agent_server/llm_utils.py` | LLM client helper: model selection, API key resolution, chat completion wrapper |
| `agent_server/synthesizer.py` | Cross-sectional synthesis of analysis results (assessment, celebration points, quick wins) |
| `agent_server/error_analysis.py` | Hybrid auto-labeling: programmatic SQL comparison + LLM fallback for ambiguous cases |
| `agent_server/genie_creator.py` | Creates new Genie Spaces via Databricks API with config validation and constraints |
| `agent_server/ingest.py` | Databricks SDK wrapper for fetching Genie Space configs |
| `agent_server/models.py` | Pydantic models: `AgentInput`, `AgentOutput`, `Finding`, `SectionAnalysis`, `ChecklistItem`, `OptimizationSuggestion`, `OptimizationRequest/Response`, `ConfigMergeRequest/Response` |
| `agent_server/prompts.py` | LLM prompt templates for checklist evaluation and optimization suggestions |
| `agent_server/sql_executor.py` | SQL execution via Databricks Statement Execution API |
| `src/App.tsx` | Main React app orchestrating both Analyze and Optimize modes |
| `src/components/*Phase.tsx` | Analyze mode UI: `InputPhase`, `IngestPhase`, `AnalysisPhase`, `SummaryPhase` |
| `src/components/BenchmarksPage.tsx` | Optimize mode: Select benchmark questions for labeling |
| `src/components/LabelingPage.tsx` | Optimize mode: Review auto-labels and override with manual labels |
| `src/components/FeedbackPage.tsx` | Optimize mode: Review labeling session summary with auto-label stats |
| `src/components/OptimizationPage.tsx` | Optimize mode: Display AI-generated optimization suggestions with selection |
| `src/components/PreviewPage.tsx` | Optimize mode: Side-by-side JSON diff of selected config changes |
| `src/components/SuggestionCard.tsx` | Card component showing field path, rationale, and current vs. suggested values |
| `src/hooks/useAnalysis.ts` | Main state hook managing both Analyze and Optimize workflows |
| `src/hooks/useTheme.ts` | Theme hook: system preference detection, localStorage persistence |
| `src/components/ScoreGauge.tsx` | Animated radial SVG progress gauge for compliance scores |
| `src/components/DataTable.tsx` | Reusable table for displaying SQL query results |

### Analysis Approach

All checklist items are defined in `docs/checklist-by-schema.md` and evaluated by the LLM. The markdown file is parsed at runtime, so users can add/remove/modify checks without changing code.

### Optimization Approach

The Optimize mode follows a 5-step workflow:
1. **Benchmarks**: Select benchmark questions from the Genie Space config
2. **Labeling**: For each question, query Genie for SQL, execute it, auto-label via hybrid error analysis (programmatic + LLM), and allow user override
3. **Feedback**: Review the labeling session summary
4. **Optimization**: AI analyzes the config + labeling feedback to generate field-level suggestions (field path, current value, suggested value, rationale, priority)
5. **Preview**: Select suggestions and view a side-by-side JSON diff of the proposed config changes (programmatic merge, no LLM needed)

## Key Patterns

- **Markdown-driven checklist**: `docs/checklist-by-schema.md` is parsed at runtime; all items evaluated by LLM
- **MLflow Tracing**: All LLM calls traced with session grouping via `mlflow.start_span()`
- **Streaming SSE**: `predict_streaming()` yields progress updates; frontend consumes via `/api/analyze/stream`. Events have `status` field: `fetching`, `analyzing` (with `current`/`total`), `complete`, `result`
- **OBO Auth**: In Databricks Apps, `OBOAuthMiddleware` extracts `x-forwarded-access-token` and stores it in a `contextvars.ContextVar`. All downstream code (`get_workspace_client()`, `get_llm_api_key()`) reads from this ContextVar. Locally, the token is always `None` and falls back to PAT/CLI auth.
- **ContextVar propagation gotcha**: Starlette runs sync generators in a new thread, and `ThreadPoolExecutor` spawns its own threads — the OBO token ContextVar must be explicitly captured before and re-set inside these contexts. See `stream_analysis`, `invoke_stream` (capture + re-set), and `stream_optimizations` (`contextvars.copy_context().run()`).
- **Section Constants**: The 10 analyzed sections are defined in `SECTIONS` list in `agent.py`

## Environment Configuration

Configuration in `.env.local` (created by quickstart.sh):

```bash
DATABRICKS_HOST=https://your-workspace.cloud.databricks.com
DATABRICKS_CONFIG_PROFILE=DEFAULT   # or DATABRICKS_TOKEN for PAT
MLFLOW_EXPERIMENT_ID=<id>           # Optional: set to enable tracing
LLM_MODEL=databricks-claude-sonnet-4-6
SQL_WAREHOUSE_ID=<id>               # Optional fallback: users can enter in UI
GENIE_TARGET_DIRECTORY=<path>       # Optional fallback: users can enter in UI
```

Notes:
- MLflow tracing is optional. Leave `MLFLOW_EXPERIMENT_ID` empty to disable it.
- `SQL_WAREHOUSE_ID` and `GENIE_TARGET_DIRECTORY` can be entered by users in the UI (under "optional settings" on the input page). Env vars serve as fallback defaults when the UI fields are left empty.

## Technology Stack

- **Backend**: Python 3.11+, FastAPI, Databricks SDK 0.38+, MLflow 3.6+
- **Frontend**: React 18, TypeScript, Vite, Tailwind CSS v4
- **Design System**: Self-hosted fonts (Cabinet Grotesk, General Sans, JetBrains Mono), CSS custom properties for theming, dark mode support
- **Frontend location**: React frontend lives at project root (`src/`, `package.json`, `vite.config.ts`); Databricks Apps auto-builds from root `package.json`
- **Package managers**: `uv` (Python), `npm` (frontend)
- **LLM**: Databricks-hosted Claude Sonnet via serving endpoints

## Frontend Design System

The frontend uses a custom design system with:

- **Theming**: CSS custom properties in `index.css` with `:root` (light) and `.dark` (dark) selectors
- **Fonts**: Self-hosted in `public/fonts/` (project root) - Cabinet Grotesk (display), General Sans (body), JetBrains Mono (code)
- **Dark Mode**: Auto-detects `prefers-color-scheme`, persists user override in localStorage (`genierx-theme` key)
- **Animations**: CSS-only animations defined in `src/index.css` (`@keyframes fadeSlideUp`, `scan`, etc.)

To add/download fonts, get them from:

- [Cabinet Grotesk](https://www.fontshare.com/fonts/cabinet-grotesk)
- [General Sans](https://www.fontshare.com/fonts/general-sans)
- [JetBrains Mono](https://www.jetbrains.com/lp/mono/)

---
> Source: [hiydavid/dbx-genie-rx](https://github.com/hiydavid/dbx-genie-rx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
