## terris-cloud-api

> This file provides foundational mandates for Gemini CLI when working on this project.

# Gemini CLI Project Context: Terris Cloud API

This file provides foundational mandates for Gemini CLI when working on this project.

## Project Overview
- **Type:** FastAPI Python API
- **Manager:** `uv`
- **Database:** Dolt (MySQL-compatible with versioning)
- **Issue Tracking:** Beads (bd)

## Development Workflow

### Dependency Management
- Always use `uv` for managing dependencies.
- To add a package: `uv add <package>`
- To sync: `uv sync`
### 🟢 Issue-First Workflow (Beads)
To ensure the most effective collaboration and persistent memory:
1. **Never Code Without an Issue:** Before starting any work, ensure an issue exists (`bd create`) and is marked `in_progress`.
2. **Issue-Commit Mapping:** Every issue created MUST result in at least one git commit. Reference the issue ID in the commit message.
3. **Use Memory Fields:**
   - Use `bd update <id> --design "..."` to document the implementation plan *before* writing code.
   - Use `bd update <id> --notes "..."` to record intermediate findings, blockers, or deviations.
4. **Break Down Complexity:** Use `bd dep add <child> <parent>` to create dependency graphs for complex features.

## 🏗️ Architectural Mandates: Layered MVC + Repository
To ensure code organization and separation of concerns, the project MUST follow a layered architecture:

1.  **Controllers (Routes):** Located in `app/api/`. Handle HTTP requests, input validation (via Schemas), and call Services. They SHOULD NOT contain business logic or direct DB access.
2.  **Services:** Located in `app/services/`. Contain core business logic. They orchestrate calls to Repositories and other services.
3.  **Repositories:** Located in `app/repositories/`. Handle all direct data access logic (Dolt/SQL queries). They abstract the persistence layer from the rest of the application.
4.  **Models:** Located in `app/models/`. Define the database schema/structure.
5.  **Schemas:** Located in `app/schemas/`. Pydantic models for JSON Request/Response validation and serialization.

### Directory Structure
```text
src/
├── app/
│   ├── api/          # Controllers (FastAPI routes)
│   ├── services/     # Business logic layer
│   ├── repositories/ # Data access layer
│   ├── models/       # Database models
│   ├── schemas/      # Pydantic models (DTOs)
│   └── core/         # Configuration, DB connection, security
└── main.py           # Application entry point
tests/                # Pytest suite
```

## ✅ Quality Control Mandates
To ensure code reliability and maintainability, the following MUST be run and pass before ANY commit:

1.  **Testing:** Use `pytest`. Every new feature or bug fix MUST include corresponding tests.
2.  **Coverage:** Aim for at least **80% code coverage**. Use `pytest-cov` to verify.
3.  **Linting & Formatting:** Use `ruff`. Run `ruff check .` and `ruff format .` to ensure style compliance.
4.  **Type Checking:** Use `mypy`. All new code SHOULD be type-hinted and pass `mypy .`.

## 🧱 Commit Hygiene & Constraints
... (previous content) ...

## Session Recovery
- **Start:** Run `bd prime` followed by `bd ready` to re-sync with the current mission.
- **End:**
    1.  **Validate:** Run `pytest`, `ruff`, and `mypy`.
    2.  **Coverage:** Ensure `pytest-cov` reports >= 80%.
    3.  **Sync Beads:** `bd dolt push`.
    4.  **Commit:** Follow the **Session Close Protocol** (Git + Beads ID).

### Database Management (Dolt)
- The Dolt SQL server is configured in `config.yaml` and typically runs on port `3307`.
- Primary DB: `terris-cloud-api`
- Beads DB: `beads_terris-cloud-api`

## 🕵️ Stealth & Local Configuration
This project uses a "local-only" setup for infrastructure to prevent leaking local environment details or internal issue tracking to GitHub.

- **Issue Tracking (Beads):** The `.beads/` directory and the `beads_terris-cloud-api/` database are ignored by git. Do not commit them.
- **Database (Dolt):** All Dolt repositories (`.dolt/`, `.doltcfg/`) and the server config (`config.yaml`) are ignored.
- **Python/uv:** Virtual environments and lock files are local-only.

Ensure that any changes to these systems are documented in `AGENTS.md` or `GEMINI.md` but keep the underlying data files excluded from version control.

## Technical Standards
- Follow FastAPI best practices (Pydantic models for request/response).
- Use `pytest` for testing (ensure tests are created for new features).
- Maintain `AGENTS.md` as the shared "brain" for all AI collaborators.

## Session Start Checklist
1. Verify Dolt SQL server is running: `ps aux | grep dolt`.
2. Get task context: `bd prime`.
3. Check for ready tasks: `bd ready`.
4. Pull latest beads: `bd dolt pull`.

## Session End Checklist
1. Review changes: `git status`.
2. Sync beads: `bd dolt push`.
3. Follow the **Session Close Protocol** provided by `bd prime`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bensoer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bensoer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
