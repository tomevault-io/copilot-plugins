## astroloh

> Astroloh is a professional astrological voice skill for Yandex Alice built with FastAPI, PostgreSQL, and Kerykeion 4.x for advanced astronomical calculations.

# Astroloh Repository Rules

Astroloh is a professional astrological voice skill for Yandex Alice built with FastAPI, PostgreSQL, and Kerykeion 4.x for advanced astronomical calculations.

## Enforcing Code Quality and Formatting

"Run black app/ tests/ && isort app/ tests/ before every commit."
"All functions must have type hints - mypy enforcement is enabled."
"Never add code comments unless explicitly requested by the user."
"Use async/await for all I/O operations - this is a FastAPI async application."

## Critical Time Handling Rules

"NEVER import datetime directly except in app/utils/astro_time_utils.py"
"Always use: from app.utils.astro_time_utils import utcnow, now, current_timestamp"
"Replace datetime.now() with utcnow() and datetime.utcnow() with utcnow()"
"Use db_timestamp_default() for SQLAlchemy default timestamps, never datetime.now()"

## Alice Voice Interface Constraints

"Horoscope responses must be maximum 800 characters for Alice TTS."
"Compatibility responses are limited to 600 characters maximum."
"Never include more than 5 buttons in Alice responses - this breaks the interface."
"All responses must complete within 5 seconds for Alice voice interface."

## Security and Data Protection

"Never log API keys, secrets, or personal astrological data in any format."
"All user datetime inputs must go through parse_birth_datetime() for security validation."
"Use encryption utilities for storing personal birth data - GDPR compliance required."
"JWT tokens must have proper expiration handling."

## Testing Requirements

"Maintain minimum 80% test coverage - use pytest --cov=app to verify."
"Run pytest -m unit for unit tests, pytest -m integration for integration tests."
"Mock all external services (Yandex GPT, astronomical backends) in tests."
"Validate Alice response constraints (length, buttons) in integration tests."

## Astronomical Calculation Patterns

"Primary backend: Kerykeion 4.x with graceful fallbacks to Swiss Ephemeris → Skyfield → Built-in."
"Always implement fallback mechanisms - users should never see calculation failures."
"Use Redis caching: natal charts (30 days TTL), transits (1 hour TTL)."
"Monitor calculation performance and cache hit rates."

## Russian Localization Requirements

"Support all 6 Russian grammatical cases for zodiac signs."
"Include TTS optimization with proper stress marks for Alice voice."
"Handle speech recognition errors in voice preprocessing."
"Use Russian astrological terminology, not direct translations."

## Pre-commit Commands (MANDATORY)

"Before any commit, run these commands in order:

1. black app/ tests/
2. isort app/ tests/
3. flake8 app/ tests/
4. mypy app/
5. pytest --cov=app"

## Environment and Deployment

"Use docker-compose up --build for development - includes PostgreSQL and Redis."
"Full container restart required when changing environment variables."
"Critical env vars: DATABASE_URL, YANDEX_API_KEY, YANDEX_FOLDER_ID, SECRET_KEY"
"Run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 for local development."

## Service Integration Patterns

"Check kerykeion_service.is_available() before using advanced astrology features."
"Implement graceful degradation: if Kerykeion fails, fallback to basic calculations."
"Use performance_monitor.start_operation() for critical calculations."
"Include service prefixes in log messages: KERYKEION_SERVICE_, AI_CONSULTATION_"

## Response Time Targets

"Cached requests: <500ms, Complex calculations: <2s, Alice interface: <5s maximum."
"Use background pre-computation for popular astrological data."
"Monitor and alert on slow operations >2000ms."

## Common Anti-Patterns to Avoid

"Never use: from datetime import datetime; use astro_time_utils instead."
"Never log user birth data or API keys in any format."
"Don't create SQLAlchemy defaults with lambda: datetime.now() - use db_timestamp_default()."
"Avoid direct astronomical calculations - use the multi-backend service pattern."

## File Structure Guidelines

"Read app/utils/astro_time_utils.py first - central time handling system."
"app/services/kerykeion_service.py contains professional astrology backend."
"app/api/yandex_dialogs.py handles Alice voice interface integration."
"CLAUDE.md has complete project documentation and context."

## Commit Guidelines

"Never commit code without explicit user request."
"Format: 'type: description' (feat, fix, refactor, docs, test, chore)"
"Test locally before committing - no broken builds allowed."

## Performance Monitoring

"Track cache hit rates, response times, and backend selection."
"Use correlation IDs for request tracking across services."
"Monitor Redis memory usage and Yandex GPT API success rates."

## Agent Permissions and Workflow

"Agent should create PRs only when explicitly requested by the user."
"Linear issue self-assignment is enabled - agent can assign issues to itself when working."
"Rules are automatically detected from .cursorrules file in repository root."
"Agent should move Linear issues to 'Started' state when beginning work."

## Model Configuration Preferences

"Use Anthropic Claude models for optimal performance on this Python/FastAPI project."
"Google Gemini is available as alternative - good for code generation and analysis."
"Complex astronomical calculations may benefit from Claude's reasoning capabilities."
"Russian language processing works well with both Claude and Gemini multilingual support."
"Cost-effectiveness is important - prefer efficient models for routine operations."

## Repository-Specific Agent Behavior

"Agent has full access to all project files including sensitive configuration."
"Environment variables should be handled carefully - never expose in logs or commits."
"Agent can run all development commands (black, pytest, docker-compose) as needed."
"Network access available for external API testing (Yandex GPT, astronomical services)."

## Working with AI Agents

"Use @claude for code implementation, debugging, and Russian language tasks."
"Use @gemini for codebase research, analysis, and information processing."
"Use @codegen in Slack/Linear for repository-wide changes and issue resolution."
"Tag agents in Slack threads to maintain conversation context."
"Be specific: 'Add type hints to this function', 'Fix failing tests in test_astrology.py'."
"For Russian tasks: 'Generate Russian horoscope text with proper grammatical cases'."
"For complex debugging: 'Analyze this multi-backend fallback issue step by step'."
"Always specify test requirements: 'Add tests with 80% coverage for this service'."
"See ORGANIZATION_AGENT_RULES.md for complete agent usage guidelines."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evgenygurin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
