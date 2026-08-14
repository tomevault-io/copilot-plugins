## easy-islanders-project

> **Backend (Django + PostgreSQL)**

# Easy Islanders - Agentic Coding Guide

## Build & Test Commands

**Backend (Django + PostgreSQL)**
- Run tests: `pytest` or `python -m pytest`
- Run single test: `pytest tests/test_basic.py::BasicTest::test_django_setup`
- Run with coverage: `pytest --cov=. --cov-report=term-missing`
- Migrations: `python manage.py migrate`
- Dev server: `python manage.py runserver` or `make runserver`
- Router eval: `python scripts/eval_router.py --corpus scripts/router_eval_corpus.json` or `make eval-router`

**Frontend (React)**
- Install: `cd frontend && npm ci`
- Dev server: `npm start`
- Build: `npm run build`
- Tests: `npm test -- --coverage --watchAll=false`
- Lint: `npm run lint`

## Code Style Guidelines

**Imports (Python)**
- Standard library first, then Django, then third-party, then local imports
- Use `from __future__ import annotations` for type hints in complex modules
- Group related imports: `from django.db import models` not multiple lines
- Example: assistant/models.py:1-7, router_service/views.py:1-8

**Type Hints**
- Use type hints for function signatures: `def route_intent(request) -> Response:`
- Use Dict, List, Optional from typing module
- Complex types in docstrings when inline hints are verbose

**Naming Conventions**
- Models: PascalCase (Request, DemandLead, ConversationThread)
- Functions/methods: snake_case (run_router, get_threads, mark_as_read)
- Constants: UPPER_SNAKE_CASE (GREETING_KEYWORDS, PHOTO_KEYWORDS)
- Private helpers: prefix with underscore (_is_greeting, _looks_like_property_search)

**Django Patterns**
- Use UUIDField for primary keys: `id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)`
- Add indexes for frequently queried fields: `models.Index(fields=['user', 'is_active'])`
- Use JSONField for flexible attributes/metadata with helpful help_text (e.g., for category grouping in UserPreference)
- Atomic operations: Use `@transaction.atomic` for critical writes
- Permissions: Define in Meta class: `permissions = [("can_approve_broadcasts", "...")]`
- Prefer metadata JSONField over extra model fields for non-unique, flexible data (see UserPreference.metadata for category storage)

**Error Handling**
- Return HTTP 400 for validation errors with descriptive messages
- Use Django's built-in validators where possible
- Log errors with context: `logger.error(f"Router failed: {err}", exc_info=True)`
- Graceful degradation: fallback chains for AI/LLM failures

**Documentation**
- Docstrings for complex functions/classes (see Message model:216-228, views_messages.py:1-14)
- Inline comments for non-obvious logic (business rules, performance hacks)
- Architectural mandates in model docstrings (Q20, Q4a references in Message model)
- API contracts documented in API_CONTRACTS.md

**Testing**
- Tests in tests/ directory following test_*.py pattern
- Use Django TestCase for DB-backed tests
- Mock external services (Twilio, OpenAI) in tests
- Fixtures in conftest.py for shared test data

---
> Source: [templeobijnr/easy_islanders_project](https://github.com/templeobijnr/easy_islanders_project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
