## aegis-stack

> Purpose: Guide Copilot to produce code aligned with Aegis Stack architecture and conventions.

# GitHub Copilot Instructions for Aegis Stack

Purpose: Guide Copilot to produce code aligned with Aegis Stack architecture and conventions.

## Core Architectural Truths
- Components (app/components/*) define capability + orchestration (backend, frontend, scheduler, etc.).
- Services (app/services/*) contain pure business logic. No routes, jobs, hooks, or infra config.
- Components call services. Services never import components.
- Do NOT call components “services.” Never generate names like scheduler_service. Use scheduler component.
- Async-first: prefer async functions where I/O occurs (FastAPI endpoints, scheduler jobs, integrations).
- Explicit > magic: always import what you use; no hidden auto-discovery except documented backend hook folders.

## When Generating Code
Prefer small, focused, testable functions. Keep side effects in components; keep computation in services.

### Services Pattern
- Pure logic, stateless except via parameters.
- Return data structures (dict / pydantic model), not framework responses.
- Place in app/services/<domain>_service.py.
- Example function signature pattern:
```python
async def generate_report(user_id: str) -> ReportResult: ...
```

### Component Additions
Backend:
- Add FastAPI routes under app/components/backend/api/.
- Register routers in routing.py (do not instantiate new FastAPI app).
Frontend:
- Place session/page logic in app/components/frontend/.
Scheduler:
- Add jobs in scheduler component factory (create_scheduler()) using scheduler.add_job().
- Do not put APScheduler setup in services/.

### Scheduler Jobs
- Define business logic in services.
- In component: import service function, schedule with clear id, name, and reasonable trigger.
- Use max_instances=1 for idempotent recurring jobs unless concurrency required.
- Use coalesce=True for periodic tasks where catching up is safe.

### Logging
- Use structlog logger from app.core.log (logger).
- Prefer structured context (logger.info("msg", key=value)) if existing pattern allows; otherwise concise message strings.

### Configuration
- Import from app.core.config import settings.
- Do NOT re-instantiate Settings.
- Feature flags / tunables exposed as settings attributes.

### DRY Enforcement
Before generating code that repeats logic (file paths, error handling, settings access, logging setup), search for existing utilities in:
- app/core/
- existing services modules.

### Error Handling
- Raise domain-specific exceptions (create them in a local exceptions module) rather than broad Exception.
- Avoid swallowing exceptions silently; log and re-raise or convert to meaningful error.

### Type Safety
- Include type hints everywhere.
- For async functions returning nothing: -> None.
- Prefer Pydantic models for structured payloads crossing component boundaries.

## Testing Guidance
- Place tests in tests/ mirroring module structure.
- Async tests: use pytest.mark.asyncio.
- For services: isolate by mocking external integrations (not yet defined? leave TODO).
- For scheduler jobs: test underlying service function, not the scheduling loop.

## Documentation Prompts
When adding a new capability:
1. What capability (not just implementation)?
2. Why this implementation vs alternatives?
3. Integration points (hooks, entrypoints, jobs).
4. Swapping guidance (what would change).

## Naming Conventions
- snake_case for modules/functions.
- <domain>_service.py for service modules.
- Avoid suffix “util” unless generic; prefer descriptive noun/verb.
- Jobs: id and name human-readable (e.g., "system_status_check", "System Health Check").

## Anti-Patterns (Reject / Refactor)
- Putting job definitions inside services.
- Creating new FastAPI app instances inside routes.
- Direct environment variable reads outside config module.
- Copy-pasted logging or setup code.
- Calling components “microservices” or “services.”

## Prompts to Bias Copilot
When extending code, think:
- Can this logic live in a service function?
- Does a component already handle orchestration?
- Is there existing shared utility?
- Are type hints and error paths complete?

## Example: Adding a New Scheduled Job
Service (business logic):
```python
# app/services/cleanup_service.py
from datetime import datetime

async def purge_expired_sessions(now: datetime) -> int:
    """
    Remove expired sessions. Return count removed.
    Pure logic; no scheduler or logging side effects besides minimal info if needed.
    """
    # ...implementation...
    return removed
```
Scheduler component modification:
```python
# app/components/scheduler/main.py (inside create_scheduler)
from app.services.cleanup_service import purge_expired_sessions
scheduler.add_job(
    purge_expired_sessions,
    trigger="cron",
    minute="*/30",
    id="purge_expired_sessions",
    name="Purge Expired Sessions",
    max_instances=1,
    coalesce=True,
)
```

## Example: Adding Backend Route
```python
# app/components/backend/api/routes/session.py
from fastapi import APIRouter
from app.services.cleanup_service import purge_expired_sessions
from datetime import datetime

router = APIRouter(prefix="/sessions", tags=["sessions"])

@router.post("/purge")
async def purge():
    removed = await purge_expired_sessions(datetime.utcnow())
    return {"removed": removed}
```
Register router in routing.py (do not create new FastAPI app).

## Make & Tooling
- After code generation or edits: run make check (ruff + mypy + pytest).
- Maintain zero warnings baseline.

## Commit Discipline
- Keep changes atomic: one capability or refactor per commit.
- Update docs if a new capability is introduced.

## Output Expectations
Copilot should:
- Suggest focused, typed, async-aware functions.
- Avoid generating large monolithic modules.
- Preserve architectural boundaries (components vs services).
- Include minimal but clear docstrings (one-line summary + detail if complex).

Copilot should NOT:
- Introduce new external dependencies without explicit directive.
- Autogenerate environment variable parsing outside config.
- Rename established modules or break template structure.

## Quick Reference Paths
- CLI entry: aegis/__main__.py
- Template root: aegis/templates/cookiecutter-aegis-project/
- Generated project sample component: app/components/
- Scheduler logic: app/components/scheduler/main.py
- Logging setup: app/core/log.py
- Settings: app/core/config.py
- Integration layer: app/integrations/main.py

## Style Recap
- Concise, explicit, typed, async-first, DRY, capability-focused.
- Correct terminology: COMPONENT (capability) vs SERVICE (business logic).
- Avoid “microservice” language.

End of instructions.

---
> Source: [lbedner/aegis-stack](https://github.com/lbedner/aegis-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
