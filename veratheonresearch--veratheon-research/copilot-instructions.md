## veratheon-research

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Package Manager**: This repository uses `uv` (Astral's package manager). Do NOT use `pip` or `pipenv`.

**Run the main application**:
```bash
uv run python run.py
```

**Run the FastAPI server**:
```bash
uv run python server.py
```

**Run with Docker Compose** (includes API and UI):
```bash
docker-compose up
```

**Note**: Supabase should be running separately (see Supabase setup below)

**Run tests**:
```bash
uv run pytest
```

**Run specific test file**:
```bash
uv run pytest tests/unit/historical_earnings/test_historical_earnings_util.py -v
```

**Run tests with coverage**:
```bash
uv run pytest --cov=src
```

**Install dependencies**:
```bash
uv sync
```

**UI Development** (SvelteKit frontend in `agent-ui/`):
```bash
cd agent-ui
npm run dev        # Development server
npm run build      # Production build  
npm run test       # Run tests
npm run lint       # Linting
```

## Architecture

This is a **stock research agent** for stock analysis using async flows and OpenAI Agents SDK with FastAPI backend and SvelteKit UI.

### Key Architectural Principles

1. **Strict Separation of Concerns**:
   - **Flows** (`src/flows/`): Thin wrappers for async orchestration, contain NO business logic
   - **Tasks** (`src/tasks/`): Data orchestration only, contain NO business logic  
   - **Business Logic** (`src/research/`): All core research logic lives here

2. **Data Flow Architecture**:
   - Main flow: `src/flows/research_flow.py:main_research_flow()`
   - Subflows: Historical earnings, financial statements, earnings projections, management guidance, forward PE analysis, news sentiment, trade ideas
   - Uses Alpha Vantage API for financial data (`src/lib/`)

3. **Model Management**:
   - Uses `litellm` for model abstraction (`src/lib/llm_model.py`)
   - Supports multiple Ollama models (Gemma 27B/12B/4B, GPT-OSS) and OpenAI models
   - Model selection via `MODEL_SELECTED` environment variable

4. **Full-Stack Application**:
   - **Backend**: FastAPI server (`server/api.py`) with `/health` and `/research` endpoints
   - **Frontend**: SvelteKit UI (`agent-ui/`) with Tailwind CSS and DaisyUI
   - **Infrastructure**: Docker Compose for API/UI, Supabase for caching/state management/RAG

### Research Pipeline

The agent performs comprehensive stock research through these sequential steps:
1. **Historical Earnings Analysis**: Establishes foundational baseline patterns
2. **Financial Statements Analysis**: Analyzes recent changes for projection accuracy  
3. **Independent Earnings Projections**: Creates baseline projections for consensus validation
4. **Management Guidance Analysis**: Cross-checks against management guidance from earnings calls
5. **Peer Group Analysis**: Identifies comparable companies (enhanced with financial context)
6. **Forward PE Sanity Check**: Validates earnings data quality  
7. **Forward PE Analysis**: Calculates valuation metrics (enhanced with projections and guidance)
8. **News Sentiment Analysis**: Analyzes recent news sentiment (enhanced with earnings context)
9. **Trade Ideas Generation**: Synthesizes all analyses into actionable insights

### Environment Variables

Required in `.env` file:
- `ALPHA_VANTAGE_API_KEY`: For financial data
- `OPENAI_API_KEY`: For AI model access (if using OpenAI)
- `MODEL_SELECTED`: Model choice (local_gemma27b, nord_gemma27b, local_gemma12b, nord_gemma12b, local_gemma4b, nord_gemma4b, local_gptoss, nord_gptoss, o4_mini)
- `LOCAL_OLLAMA_URL`/`NORD_OLLAMA_URL`: Ollama server URLs (when using Ollama models)

Optional in `.env` file:
- `USE_EARNINGS_ESTIMATES_API`: Set to "false" to use the legacy Earnings Calendar API instead of the new Earnings Estimates API for consensus EPS data (default: true)
- `HOST`: Server host (default: 0.0.0.0)
- `PORT`: Server port (default: 8085)
- `SUPABASE_URL`: Supabase project URL (e.g., http://127.0.0.1:54321 for local)
- `SUPABASE_ANON_KEY`: Supabase anonymous/publishable key
- `SUPABASE_SERVICE_KEY`: Supabase service role key (for server-side operations)

### Supabase Setup

The application uses Supabase for:
- **Job Tracking**: `research_jobs` table tracks research job status and metadata
- **Caching**: `research_cache` table caches analysis results with TTL via `expires_at`
- **RAG (Retrieval Augmented Generation)**: `research_docs` table stores research reports with vector embeddings
- **User History**: `user_research_history` table tracks user research activity
- **System Logs**: `system_logs` table for centralized error tracking

Required Supabase tables:
- `research_jobs` (bigint id, symbol, status, metadata jsonb, timestamps)
- `research_cache` (bigint id, cache_key unique, symbol, report_type, data jsonb, expires_at)
- `research_docs` (bigint id, content, title, embedding vector, metadata jsonb, token_count)
- `user_research_history` (bigint id, user_id, symbol, job_id, metadata jsonb)
- `system_logs` (bigint id, log_level, component, message, job_id, symbol, stack_trace)

For local development, run Supabase in a separate directory and use `supabase status` to get connection details.

**Enable Realtime for research_jobs table** (required for frontend live updates):
```sql
alter publication supabase_realtime add table research_jobs;
```

**Frontend Configuration**:
The SvelteKit UI uses Supabase Realtime for live job status updates instead of polling. Configure frontend environment variables in `agent-ui/.env`:
- `VITE_SUPABASE_URL`: Same as backend SUPABASE_URL
- `VITE_SUPABASE_ANON_KEY`: Same as backend SUPABASE_ANON_KEY

### Testing Strategy

- Uses `pytest` with path configuration in `pyproject.toml`
- Test files in `tests/` directory with unit tests in `tests/unit/`
- Mocks external API calls to avoid hitting real APIs during tests
- Supabase client is automatically mocked in all tests via `conftest.py` fixtures

## Development Guidelines

- When adding new research capabilities, put business logic in `src/research/`
- When adding new data sources, put integration code in `src/lib/`
- Keep flows and tasks as thin orchestration layers
- All new features must maintain the separation between orchestration and business logic
- Use `uv run` prefix for all Python commands
- Frontend development uses SvelteKit with TypeScript, Tailwind CSS, and DaisyUI components
- API endpoints follow REST conventions and return structured JSON responses
- All external API calls should be mocked in tests to avoid hitting real services

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VeratheonResearch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
