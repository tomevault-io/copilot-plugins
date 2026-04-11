## ai-agent-codex

> This repository is a greenfield data platform project built around `hh.ru API`, `Apache Airflow`, `PostgreSQL`, `dbt`, `Python`, `FastAPI`, `asyncio`, and `Vue.js`.

# HH Data Platform Project

This repository is a greenfield data platform project built around `hh.ru API`, `Apache Airflow`, `PostgreSQL`, `dbt`, `Python`, `FastAPI`, `asyncio`, and `Vue.js`.

## Primary Goal

Build a maintainable analytics platform that ingests job-market data from HH API, stores raw data in PostgreSQL, transforms it with dbt, and exposes clean analytical marts plus product-facing APIs and dashboards for reporting and realtime market monitoring.

## Project Priorities

1. Requirements before implementation.
2. Small, reviewable changes over broad rewrites.
3. Clear separation between ingestion, storage, transformation, and analytics.
4. Idempotent pipelines and reproducible local development.
5. Project documentation kept close to code and updated with each material change.

## Expected Repository Structure

- `dags/` Airflow DAG definitions only.
- `src/` Shared Python code used by DAGs and data pipelines.
- `dbt/` dbt project, models, and profile templates.
- `sql/` bootstrap SQL for local PostgreSQL and raw-layer objects.
- `apps/api/` FastAPI services and realtime delivery interfaces when introduced.
- `apps/web/` Vue.js frontend applications and dashboard UI when introduced.
- `docs/specs/` short functional and technical specs.
- `.opencode/agents/` role-based subagents.
- `.opencode/skills/` reusable domain skills.

## Architecture Rules

### Data Flow

Use this logical flow unless a task explicitly requires a different design:

`hh api -> landing/raw -> postgres raw tables -> dbt staging -> dbt marts`

For product-facing delivery, expose analytics through a thin API and frontend layer:

`dbt marts -> fastapi services -> vue dashboards`

### Layer Responsibilities

- `raw` keeps source fidelity and ingestion metadata.
- `staging` standardizes names, types, and basic cleaning.
- `marts` contains business-facing analytical models.
- `api` exposes application and realtime analytics contracts without reimplementing warehouse logic.
- `web` presents dashboards, filters, and visual analytics for end users.

### Pipeline Standards

- New pipelines must be safe to re-run.
- Preserve source identifiers from HH API.
- Store ingestion timestamps and load dates in raw objects.
- Prefer append-only raw ingestion unless deduplication rules are explicitly defined.
- Every production-intended pipeline should define retry policy, owner, and schedule.

### Modeling Standards

- Prefer explicit column lists over `select *` in dbt models unless there is a strong reason not to.
- Cast key analytical fields in staging models.
- Keep business logic in marts, not in raw ingestion.
- Use descriptive model names such as `stg_hh_vacancies` and `mart_hh_vacancies_daily`.

## Role Routing

Use the dedicated subagent that best matches the task:

- `project-manager` for scope definition, backlog decomposition, priorities, delivery checkpoints, and definition of done.
- `system-analyst` for requirements, source-to-target mapping, contracts, assumptions, and specification documents.
- `data-analyst` for metric design, analytical questions, KPI definition, and reporting requirements.
- `data-engineer` for Airflow, dbt, SQL, Python ETL, and ingestion implementation.
- `infrastructure-engineer` for Docker Compose, runtime setup, environment configuration, and local platform operations.
- `backend-engineer` for FastAPI, asyncio services, analytics APIs, streaming delivery, and backend integration over platform data.
- `frontend-engineer` for Vue.js dashboards, filtering UX, realtime widgets, and analytical data visualization.

## Delivery Workflow

Default sequence for net-new work:

1. Clarify scope and success criteria.
2. Capture requirements or assumptions in `docs/specs/` when the task affects architecture or data contracts.
3. Implement ingestion or transformation changes in the smallest useful slice.
4. Add API and frontend slices only after the analytical contract is clear enough to expose.
5. Keep local runtime config aligned with code changes.
6. Update docs when structure, interfaces, or operating steps change.

## Documentation Expectations

- Put durable design decisions in `docs/specs/`.
- Keep README focused on setup and workflow.
- Keep agent prompts and skills in English for system-level consistency.
- Domain documentation may be written in Russian when that improves team usability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivanshamaev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ivanshamaev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
