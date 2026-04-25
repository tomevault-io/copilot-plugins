## telenews

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Development Commands

### Running Services
Tel-Insights is a microservices system. Start services using the service runner:

```bash
# Service runner (recommended)
python run_service.py aggregator        # Telegram message collector
python run_service.py ai-analysis       # AI processing with Gemini
python run_service.py smart-analysis    # MCP server and alerts
python run_service.py alerting          # Telegram bot interface

# Alternative scripts
./run_service.sh <service>    # Linux/macOS
run_service.bat <service>     # Windows
```

### Testing
```bash
# Run all tests
pytest

# Run by category
pytest -m unit
pytest -m integration
pytest -m e2e

# With coverage
pytest --cov=src --cov-report=html

# Single test file
pytest tests/unit/test_models.py
```

### Database Management
```bash
# Apply migrations
alembic upgrade head

# Create new migration
alembic revision --autogenerate -m "Description"

# Rollback
alembic downgrade -1
```

### Code Quality
```bash
# Format code
black src/
isort src/

# Lint
flake8 src/
mypy src/
```

## Architecture Overview

Tel-Insights is a **microservices architecture** with **asynchronous message queue communication**:

```
Aggregator → RabbitMQ → AI Analysis → PostgreSQL
    ↓                        ↓            ↑
PostgreSQL              Smart Analysis ←──┘
    ↑                        ↓
Alerting ←─────────────── MCP Tools
```

### Core Services
1. **Aggregator** (`src/aggregator/`): Telegram client using Telethon, monitors channels, publishes to queue
2. **AI Analysis** (`src/ai_analysis/`): Queue consumer, processes messages with Gemini LLM, stores metadata
3. **Smart Analysis** (`src/smart_analysis/`): MCP server, frequency-based alerts, news summarization
4. **Alerting** (`src/alerting/`): Telegram bot using python-telegram-bot (TODO: not yet implemented)

### Shared Components
- **Database** (`src/shared/database.py`): PostgreSQL with SQLAlchemy, JSONB for AI metadata
- **Models** (`src/shared/models.py`): SQLAlchemy ORM models
- **Config** (`src/shared/config.py`): Environment-based configuration
- **Messaging** (`src/shared/messaging.py`): RabbitMQ queue utilities
- **Logging** (`src/shared/logging.py`): Structured logging with Rich/Structlog

## Key Technical Details

### Data Flow
1. Aggregator monitors Telegram channels → publishes events to RabbitMQ
2. AI Analysis consumes events → calls Gemini API → stores metadata in PostgreSQL
3. Smart Analysis queries database → triggers alerts → notifies users

### Database Schema
- **messages**: Core table with `ai_metadata` JSONB column (GIN indexed)
- **channels**: Monitored Telegram channels
- **media**: Deduplicated files (SHA256 hashing)
- **users/alert_configs**: User preferences and alert rules

### Environment Setup
Copy `config.env.template` to `.env` and configure:
- `DATABASE_URL`: PostgreSQL connection
- `RABBITMQ_URL`: Message queue
- `TELEGRAM_API_ID/HASH`: Telegram API credentials
- `TELEGRAM_BOT_TOKEN`: Bot token
- `GOOGLE_API_KEY`: Gemini API key
- `MONITORED_CHANNELS`: Comma-separated channel list

### Critical Dependencies
- **Telethon**: Telegram user client (MTProto protocol)
- **python-telegram-bot**: Bot framework (not yet used)
- **FastMCP**: MCP server framework for LLM tools
- **SQLAlchemy**: ORM with PostgreSQL JSONB support
- **Google Generative AI**: Gemini LLM integration

## Development Notes

### Service Communication
Services communicate **asynchronously** via RabbitMQ. Never use direct HTTP calls between core services - use the message queue to maintain loose coupling and fault tolerance.

### AI Metadata Structure
Messages store rich AI analysis in PostgreSQL JSONB:
```json
{
  "summary": "Brief message summary",
  "topics": ["technology", "AI"],
  "sentiment": "positive",
  "entities": {"organizations": ["OpenAI"], "locations": ["SF"]},
  "keywords": ["AI", "breakthrough"],
  "confidence_score": 0.95
}
```

### MCP Server
Smart Analysis runs an MCP server on port 8003 with tools:
- `summarize_news`: Generate news summaries
- `topic_trends`: Analyze topic frequency
- `check_alerts`: Process alert conditions

### Testing Structure
- `tests/unit/`: Isolated component tests
- `tests/integration/`: Service interaction tests
- `tests/e2e/`: Full pipeline tests
- Uses pytest fixtures for database, mocks, and sample data

When running linting/formatting, always use the commands specified above. The project uses Black with 88-character line length and isort with Black profile.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yishaik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
