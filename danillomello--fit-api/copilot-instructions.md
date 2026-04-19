## fit-api

> > Java 21 | Spring Boot 3.2 | PostgreSQL 15 | Gradle multi-module

# Connecthealth - fit-api

> Java 21 | Spring Boot 3.2 | PostgreSQL 15 | Gradle multi-module

## Context

This is the **backend API** for Connecthealth platform.
Shared documentation lives in the **fit-common** GitHub repo: `DanilloMello/fit-common`.

## How to start any task

1. **Run `/fit-api-overview`** — the master index skill that orchestrates everything:
   - Fetches coding standards and doc registry from fit-common
   - Points to the right spec files (architecture, patterns, testing, rules) based on your task

2. **Commands**:
   ```bash
   ./gradlew bootRun                    # Run app
   ./gradlew test                       # Run tests
   docker compose up -d                 # Start PostgreSQL
   docker compose down                  # Stop PostgreSQL
   ```

## API Contract Rule — MANDATORY

Before **adding, modifying, or removing any endpoint**, fetch the source of truth:

```
Tool: mcp__github__get_file_contents
  owner: DanilloMello
  repo: fit-common
  path: docs/API_REGISTRY.md
```

- **New endpoint** → it must be listed in Section 3 (Planned). Move it to Section 2 (Implemented) in `fit-common` after shipping.
- **Changing a contract** (path, method, request/response shape) → update `fit-common` `API_REGISTRY.md` in the same PR.
- **Removing an endpoint** → add it to Section 4 (Removed) in `fit-common`.

Never let fit-api diverge silently from the registry.

## Production Migration Doc Rule

**After every task**, check if any changes you made introduce or resolve technical debt
relevant to a production deployment. If so, update `docs/PRODUCTION_MIGRATION.md`:

- **New debt found** → add it under the appropriate severity section with current state,
  fix description, and affected files.
- **Debt resolved** → mark the item as done or remove it; update the go-live checklist.
- **Config/env var changed** → update the Environment Variables Reference table.

This keeps the doc as a living, always-current migration guide.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DanilloMello) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
