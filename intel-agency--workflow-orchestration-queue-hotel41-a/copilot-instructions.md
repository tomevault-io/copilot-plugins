## workflow-orchestration-queue-hotel41-a

> OS-APOW (Open Source Agentic Platform for Orchestration of Work) is a **Headless Agentic Orchestration Platform** that transforms interactive AI coding into autonomous background service. It converts GitHub Issues into "Execution Orders" that AI agents fulfill without human intervention, moving the agent from a passive co-pilot role to a background production service.

# AGENTS.md

## Project Overview

OS-APOW (Open Source Agentic Platform for Orchestration of Work) is a **Headless Agentic Orchestration Platform** that transforms interactive AI coding into autonomous background service. It converts GitHub Issues into "Execution Orders" that AI agents fulfill without human intervention, moving the agent from a passive co-pilot role to a background production service.

### Key Technologies

| Category | Technology | Version |
|----------|------------|---------|
| **Language** | Python | 3.12+ |
| **Package Manager** | uv | Latest |
| **Web Framework** | FastAPI | 0.115+ |
| **ASGI Server** | Uvicorn | 0.34+ |
| **Validation** | Pydantic | 2.10+ |
| **HTTP Client** | HTTPX | 0.28+ |
| **Testing** | pytest | 8.3+ |
| **Linting** | ruff | 0.8+ |
| **Type Checking** | mypy | 1.13+ |
| **Container** | Docker | Latest |

### Core Components

The system consists of four core pillars:

1. **The "Ear" (Notifier Service)** - FastAPI webhook receiver with HMAC SHA256 validation
2. **The State (Work Queue)** - GitHub Issues as database with label-based state machine
3. **The "Brain" (Sentinel Orchestrator)** - Async polling service for task management
4. **The "Hands" (Worker)** - DevContainer-based AI execution

---

## Setup Commands

### Prerequisites

- Python 3.12 or higher
- [uv](https://docs.astral.sh/uv/) package manager
- Docker (optional, for containerized deployment)
- GitHub token with appropriate permissions

### Installation

```bash
# Install dependencies with uv (recommended)
uv sync

# Or with pip
pip install -e .
```

### Environment Configuration

Set the required environment variables:

```bash
# Required
export GITHUB_TOKEN="your-github-token"
export GITHUB_ORG="your-org"
export GITHUB_REPO="your-repo"

# Optional
export SENTINEL_BOT_LOGIN="your-bot-login"  # For assign-then-verify locking
export WEBHOOK_SECRET="your-webhook-secret"  # For notifier service
```

### Running the Services

#### Sentinel Orchestrator (Polling Service)

```bash
# Using uv
uv run sentinel

# Or directly
python -m src.orchestrator_sentinel
```

#### Notifier Service (Webhook Receiver)

```bash
# Using uv
uv run notifier

# Or with uvicorn directly
uvicorn src.notifier_service:app --host 0.0.0.0 --port 8000
```

#### Using Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

---

## Project Structure

```
workflow-orchestration-queue/
├── src/
│   ├── __init__.py
│   ├── orchestrator_sentinel.py      # Background polling and dispatch service
│   ├── notifier_service.py           # FastAPI webhook ingestion
│   ├── models/
│   │   ├── __init__.py
│   │   ├── work_item.py              # Unified WorkItem model with credential scrubbing
│   │   └── github_events.py          # GitHub webhook payload schemas
│   └── queue/
│       ├── __init__.py
│       └── github_queue.py           # ITaskQueue ABC + GitHubQueue implementation
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_work_item.py         # Unit tests for WorkItem model
│   │   └── test_github_queue.py      # Unit tests for GitHubQueue
│   └── integration/
│       └── test_integration.py       # Integration tests
├── scripts/
│   ├── devcontainer-opencode.sh      # Core shell bridge for worker execution
│   └── gh-auth.ps1                   # GitHub App auth sync
├── docs/                              # Architecture and planning docs
├── plan_docs/                         # Planning documents
├── pyproject.toml                     # Project configuration and dependencies
├── Dockerfile                         # Application container definition
├── docker-compose.yml                 # Multi-service orchestration
└── README.md                          # Project documentation
```

### Key Files

| Path | Purpose |
|------|---------|
| `src/orchestrator_sentinel.py` | Sentinel orchestrator main entry point |
| `src/notifier_service.py` | FastAPI webhook receiver |
| `src/models/work_item.py` | Unified data model with credential scrubbing |
| `src/models/github_events.py` | GitHub webhook payload schemas |
| `src/queue/github_queue.py` | GitHub-backed task queue implementation |
| `pyproject.toml` | Project configuration, dependencies, and tool settings |
| `Dockerfile` | Application container definition |
| `docker-compose.yml` | Multi-service orchestration |

---

## Code Style

### Python Conventions

- **Python Version**: Target Python 3.12+
- **Line Length**: Maximum 100 characters (enforced by ruff)
- **Type Hints**: Use strict type checking with mypy
- **Imports**: Use absolute imports from `src` package
- **Docstrings**: Use triple-quoted docstrings for modules, classes, and public functions

### Linting and Formatting

The project uses **ruff** for both linting and formatting:

```bash
# Check for linting issues
uv run ruff check .

# Auto-fix linting issues
uv run ruff check --fix .

# Format code
uv run ruff format .

# Check formatting without changes
uv run ruff format --check .
```

### Type Checking

The project uses **mypy** with strict mode:

```bash
# Type check the source code
uv run mypy src/
```

### Enabled Ruff Rules

The following ruff rule categories are enabled:
- **E**: pycodestyle errors
- **W**: pycodestyle warnings
- **F**: Pyflakes
- **I**: isort
- **B**: flake8-bugbear
- **C4**: flake8-comprehensions
- **UP**: pyupgrade
- **ARG**: flake8-unused-arguments
- **SIM**: flake8-simplify
- **TCH**: flake8-type-checking
- **PTH**: flake8-use-pathlib
- **ERA**: eradicate
- **RUF**: Ruff-specific rules

### Code Style Guidelines

1. **Use Pydantic models** for data validation and serialization
2. **Use async/await** for I/O-bound operations (HTTP requests, database calls)
3. **Use type hints** for all function parameters and return values
4. **Follow PEP 8** naming conventions
5. **Keep functions focused** - each function should do one thing well
6. **Write docstrings** for public modules, classes, and functions
7. **Use f-strings** for string formatting
8. **Prefer composition over inheritance**

---

## Testing Instructions

### Running Tests

```bash
# Run all tests
uv run pytest

# Run with verbose output
uv run pytest -v

# Run with coverage report
uv run pytest --cov=src --cov-report=html

# Run only unit tests
uv run pytest -m unit

# Run only integration tests
uv run pytest -m integration

# Run a specific test file
uv run pytest tests/unit/test_work_item.py

# Run a specific test function
uv run pytest tests/unit/test_work_item.py::test_work_item_creation
```

### Test Organization

- **Unit tests**: Located in `tests/unit/` - test individual components in isolation
- **Integration tests**: Located in `tests/integration/` - test component interactions
- **Test markers**: Use `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.slow`

### Test Conventions

1. **Name test files** with `test_` prefix (e.g., `test_work_item.py`)
2. **Name test functions** with `test_` prefix (e.g., `test_work_item_creation`)
3. **Use descriptive names** that explain what is being tested
4. **One assertion per test** when practical - use multiple tests for multiple scenarios
5. **Use fixtures** for common setup and teardown
6. **Mock external dependencies** (HTTP requests, file I/O, etc.)
7. **Always add or update tests** for changed code

### Test Configuration

Test configuration is in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
asyncio_default_fixture_loop_scope = "function"
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
```

---

## Architecture Notes

### State Machine

The system uses GitHub Issue labels as a state machine:

```
┌─────────────────┐
│  agent:queued   │ ← Initial state (Notifier adds this)
└────────┬────────┘
         │ Sentinel claims task
         ▼
┌─────────────────┐
│agent:in-progress│ ← Sentinel processing
└────────┬────────┘
         │
    ┌────┴────┬─────────────┬──────────────┐
    ▼         ▼             ▼              ▼
┌───────┐ ┌───────┐ ┌───────────────┐ ┌──────────────┐
│success│ │ error │ │infra-failure │ │stalled-budget│
└───────┘ └───────┘ └───────────────┘ └──────────────┘
 Terminal  Terminal    Terminal         Terminal
```

### Label Reference

| Label | State | Description |
|-------|-------|-------------|
| `agent:queued` | Waiting | Task validated, awaiting Sentinel |
| `agent:in-progress` | Claimed | Sentinel has claimed the issue |
| `agent:reconciling` | Recovery | Stale task being recovered |
| `agent:success` | Complete | Terminal success state |
| `agent:error` | Failed | Technical failure occurred |
| `agent:infra-failure` | Infra Error | Container/build failure |
| `agent:stalled-budget` | Budget | Cost threshold exceeded |

### Key Design Patterns

1. **Markdown as Database**: GitHub Issues serve as the persistent state store
2. **Label-based State Machine**: Issue labels represent task states
3. **Assign-then-Verify Locking**: Distributed locking via GitHub issue assignments
4. **Credential Scrubbing**: Automatic sanitization of secrets before public posting
5. **Async-First**: All I/O operations use async/await for concurrency

### Credential Scrubbing

The system automatically scrubs secrets from output before posting to GitHub:

- GitHub PATs (`ghp_*`, `ghs_*`, `gho_*`, `github_pat_*`)
- Bearer tokens
- OpenAI-style API keys (`sk-*`)
- ZhipuAI keys

**Important**: When writing tests for credential-scrubbing utilities, use obviously synthetic values that will not trigger secret detection tools (e.g., `FAKE-KEY-FOR-TESTING-00000000`). Never use prefixes that match real provider formats.

---

## PR and Commit Guidelines

### Commit Message Format

Use conventional commit format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```
feat(queue): add claim timeout handling

Add timeout parameter to claim_task method to prevent
indefinite waiting during distributed lock acquisition.

Closes #123
```

### PR Title Format

Use the same format as commit messages:

```
<type>(<scope>): <subject>
```

### Before Committing

1. **Run linting**: `uv run ruff check .`
2. **Run formatting**: `uv run ruff format .`
3. **Run type checking**: `uv run mypy src/`
4. **Run tests**: `uv run pytest`
5. **Check coverage**: `uv run pytest --cov=src`

### Branch Naming

Use descriptive branch names with prefixes:

- `feat/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Test additions/updates
- `chore/` - Maintenance tasks

Example: `feat/add-webhook-retry-logic`

---

## Common Pitfalls

### Environment Variables

- **Issue**: Services fail to start with authentication errors
- **Solution**: Ensure `GITHUB_TOKEN`, `GITHUB_ORG`, and `GITHUB_REPO` are set

### Async Context

- **Issue**: Using synchronous HTTP clients in async functions
- **Solution**: Always use `httpx.AsyncClient` in async functions

### Type Checking

- **Issue**: mypy reports errors on Pydantic models
- **Solution**: Ensure `pydantic.mypy` plugin is configured in `pyproject.toml`

### Test Fixtures

- **Issue**: Tests fail due to missing async event loop
- **Solution**: Use `pytest-asyncio` with `asyncio_mode = "auto"`

### Credential Scrubbing

- **Issue**: Tests trigger secret detection tools
- **Solution**: Use synthetic test values that don't match real secret patterns

### Docker Networking

- **Issue**: Services can't communicate in Docker Compose
- **Solution**: Use service names as hostnames (e.g., `http://notifier:8000`)

### GitHub API Rate Limits

- **Issue**: API calls fail with 403 status
- **Solution**: Implement exponential backoff and respect rate limit headers

---

## Development Workflow

### Local Development

1. **Clone and setup**:
   ```bash
   git clone <repository-url>
   cd workflow-orchestration-queue
   uv sync
   ```

2. **Create a feature branch**:
   ```bash
   git checkout -b feat/my-feature
   ```

3. **Make changes and test**:
   ```bash
   # Make your changes
   uv run pytest
   uv run ruff check .
   uv run mypy src/
   ```

4. **Commit and push**:
   ```bash
   git add .
   git commit -m "feat(scope): description"
   git push origin feat/my-feature
   ```

5. **Create a pull request**

### Debugging

- **Enable debug logging**: Set `LOG_LEVEL=DEBUG` environment variable
- **Check service health**: Access `http://localhost:8000/health` for notifier service
- **View API docs**: Access `http://localhost:8000/docs` for Swagger UI
- **Check container logs**: Use `docker-compose logs -f` for containerized services

---

## Additional Resources

- [Architecture Guide](plan_docs/OS-APOW%20Architecture%20Guide%20v3.2.md)
- [Development Plan](plan_docs/OS-APOW%20Development%20Plan%20v4.2.md)
- [Implementation Spec](plan_docs/OS-APOW%20Implementation%20Specification%20v1.2.md)
- [Tech Stack](plan_docs/tech-stack.md)
- [Architecture Summary](plan_docs/architecture.md)
- [Repository Summary](.ai-repository-summary.md)
- [Main README](README.md)

---

## API Documentation

When the notifier service is running, access the auto-generated API documentation:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **Health Check**: http://localhost:8000/health

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intel-agency) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
