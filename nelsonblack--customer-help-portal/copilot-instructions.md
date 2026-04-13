## customer-help-portal

> - `frontend-analog-meter-project/` — Angular 21 frontend (port 4200), consumes both backends

# Workspace — Analog Water / Real Estate

## Projects
- `frontend-analog-meter-project/` — Angular 21 frontend (port 4200), consumes both backends
- `analog-meter-project/` — Water Billing backend, NestJS (port 3332)
- `realator_backend/` — Real Estate backend, NestJS (port 3333)

## Cross-Stack Workflow
Backend entity changes → `yarn generate` (backend) → update `.graphql` (frontend) → `npm run generate:all` (frontend)

## Database Map
- Water Billing: `analogwater_prod_db` on port 5434
- Real Estate: `realator_local_db` on port 5444

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nelsonBlack)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nelsonBlack)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
