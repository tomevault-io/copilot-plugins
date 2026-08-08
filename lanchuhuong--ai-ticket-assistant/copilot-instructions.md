## ai-ticket-assistant

> - Line length: 100 characters (Black, isort, Ruff configured in pyproject.toml)

# Code Style
- Line length: 100 characters (Black, isort, Ruff configured in pyproject.toml)
- Use `uv run black app/ tests/` and `uv run isort app/ tests/` before committing

# Architecture Decisions
- **Stateless services**: All session state in MongoDB, not memory. Any instance handles any request.
- **Dynamic schema**: Fields defined in `app/backend/config/prompt.prompty`, not code. Change prompt file to change collected fields.

# Testing
- Unit tests mock all dependencies - no MongoDB needed
- E2E tests (`test_e2e.py`) need MongoDB running on localhost:27017
- Integration tests need `RUN_INTEGRATION_TESTS=true` + API key in `.env`
- Tests use dynamic fixtures from `prompt.prompty` - changing schema auto-updates test expectations


# Environment Variables
Required (one of): `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, or `GOOGLE_API_KEY`
db: `MONGODB_URI` (default: mongodb://localhost:27017), `MONGODB_DATABASE` (default: tickets)

# Build & Test Commands
```bash
uv sync --extra dev          # Install dependencies
uv run pytest tests/ -v      # Run all tests (needs MongoDB)
uv run pytest tests/test_chat_approach.py tests/test_session_manager.py -v  # Unit tests only
./build.sh                   # Full Docker build + test
docker compose up -d         # Run app + MongoDB
```

---
> Source: [lanchuhuong/ai-ticket-assistant](https://github.com/lanchuhuong/ai-ticket-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
