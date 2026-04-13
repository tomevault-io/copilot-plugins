## mohtion

> This file provides guidance to the Gemini CLI agent when working with code in this repository.

# GEMINI.md

This file provides guidance to the Gemini CLI agent when working with code in this repository.

## Project Overview

Mohtion is a cloud-based AI agent that monitors GitHub repositories, identifies technical debt, and opens PRs with verified fixes. It runs as a GitHub App with a background worker service.

## Build and Development Commands

```bash
# Install dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Run tests with coverage
pytest --cov=mohtion

# Run linting
ruff check .

# Run type checking
mypy mohtion/

# Run the worker locally
python -m mohtion.worker

# Run the web server locally
python -m mohtion.web
```

## Architecture

### System Overview

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   GitHub     │────▶│   Mohtion    │────▶│   Mohtion    │
│   Webhooks   │     │   Web/API    │     │   Worker     │
└──────────────┘     └──────────────┘     └──────────────┘
                            │                    │
                            ▼                    ▼
                     ┌──────────────┐     ┌──────────────┐
                     │   Database   │     │   Job Queue  │
                     │  (Postgres)  │     │   (Redis)    │
                     └──────────────┘     └──────────────┘
```

### Agent Loop (Plan-Act-Verify)

```
┌─────────────────┐
│  RECONNAISSANCE │  Clone repo, run analyzers,
│    (Scanner)    │  identify tech debt targets
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   REFACTORING   │  Create branch, LLM-driven
│    (Actor)      │  code transformation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  VERIFICATION   │  Run test suite, capture logs,
│   (Verifier)    │  self-heal on failure (max 2x)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  BOUNTY CLAIM   │  Push branch, open PR via
│  (PR Creator)   │  GitHub API
└─────────────────┘
```

### Directory Structure

```
mohtion/
├── web/
│   ├── app.py              # FastAPI application
│   ├── routes/
│   │   ├── webhooks.py     # GitHub webhook handlers
│   │   └── api.py          # REST API endpoints
│   └── templates/          # Dashboard templates (optional)
├── worker/
│   ├── __main__.py         # Worker entry point
│   ├── tasks.py            # Background job definitions
│   └── scheduler.py        # Periodic scan scheduling
├── agent/
│   ├── orchestrator.py     # Main agent loop coordinator
│   ├── scanner.py          # Reconnaissance phase
│   ├── refactor.py         # Action phase
│   └── verifier.py         # Safety phase
├── analyzers/
│   ├── ast_analyzer.py     # Cyclomatic complexity, structure
│   ├── type_checker.py     # Missing type hints
│   └── duplicate.py        # Duplicate code detection
├── integrations/
│   ├── github_app.py       # GitHub App authentication
│   └── github_api.py       # GitHub API operations
├── models/
│   ├── target.py           # Tech debt target dataclass
│   ├── bounty.py           # Bounty/PR result dataclass
│   └── config.py           # Repository configuration
└── llm/
    └── client.py           # LLM API wrapper (likely wrapping Claude/Anthropic)
```

### Key Design Decisions

1. **GitHub App model** - OAuth-based installation, per-repo permissions
2. **Async worker** - Background processing with job queue (Redis/Celery or similar)
3. **Branch naming** - All branches use `mohtion/bounty-{uuid}` format
4. **Safety first** - Never open PR if tests fail; max 2 self-heal attempts
5. **Preserve behavior** - Refactoring must not change external API or functionality

### Tech Debt Detection Targets

- High cyclomatic complexity (configurable threshold)
- Missing type hints (Python) / interfaces (TypeScript)
- Deprecated library calls
- Duplicate logic across modules
- Outdated syntax patterns

## Key Constraints

- The agent MUST NOT open a PR if tests fail
- Refactoring must preserve external API and functional behavior
- Self-healing limited to 2 attempts before abandoning a target
- All git operations isolated to `mohtion/bounty-*` branches
- Rate limit PR creation (configurable max per day per repo)

## Environment Variables

```bash
GITHUB_APP_ID=           # GitHub App ID
GITHUB_PRIVATE_KEY=      # GitHub App private key (PEM)
GITHUB_WEBHOOK_SECRET=   # Webhook signature verification
ANTHROPIC_API_KEY=       # Claude API key (Current backend LLM)
DATABASE_URL=            # Postgres connection string
REDIS_URL=               # Redis connection string
```

## Session Workflow

- **TODO.md** - Check this at the start of each session for current tasks and priorities
- **LOGBOOK.md** - Update at the end of each session with a brief, descriptive summary of what was accomplished

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajaya-coder)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ajaya-coder)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
