## manager

> Rules for working on the idun_agent_manager service

# Idun Agent Manager

Read `services/idun_agent_manager/CLAUDE.md` before making changes. It contains the full API route map, auth flow, database models, and conventions.

- All DB operations are async (`AsyncSession`).
- Config stored as JSONB, validated in/out via `idun_agent_schema` Pydantic models.
- Router pattern: `_get_<resource>()` helper for fetch + 404, `_model_to_schema()` for DB model → response.
- Workspace scoping via `require_workspace` dependency on all resource endpoints.
- Migrations: `cd services/idun_agent_manager && alembic revision --autogenerate -m "description"`.
- Guardrails from the frontend use simplified configs converted via `convert_guardrail()`.

---
> Source: [Idun-Group/idun-agent-platform](https://github.com/Idun-Group/idun-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
