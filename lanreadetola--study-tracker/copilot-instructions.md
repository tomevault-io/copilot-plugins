## study-tracker

> - **Frontend**: Blazor WebAssembly (.NET 9.0), Bootstrap 5.3.3, Chart.js v4 (JS interop)

# Study Tracker Development Guidelines

## Tech Stack

- **Frontend**: Blazor WebAssembly (.NET 9.0), Bootstrap 5.3.3, Chart.js v4 (JS interop)
- **Backend**: Azure Functions v4 (.NET 8.0, isolated worker)
- **Database**: Azure Cosmos DB (NoSQL), partition key `/userId`
- **Auth**: GitHub/Microsoft OAuth via Azure Static Web Apps
- **Hosting**: Azure Static Web Apps with GitHub Actions CI/CD

## Project Structure

- `client/` — Blazor WASM frontend
- `api/` — Azure Functions API
- `specs/` — SpecKit feature specifications
- `.specify/` — SpecKit templates, scripts, constitution
- `.claude/skills/` — SpecKit slash commands

## Build & Run

```bash
# Build both projects
dotnet build

# Run locally (requires api/local.settings.json with Cosmos DB connection)
swa start http://localhost:5000 --api-location api
```

## Key Patterns

- All API endpoints authenticate via `x-ms-client-principal-id` header
- Session routes use `{id:guid}` constraint to avoid conflicts with named routes like `/stats`
- Chart.js interop is in `client/wwwroot/js/charts.js` — functions: `renderBarChart`, `renderDonutChart`, `renderLineChart`
- Mobile responsiveness uses `d-none d-md-block` / `d-block d-md-none` to switch between table and card layouts
- Touch targets: 44px min-height on mobile via `.btn-mobile` class

## Conventions

- Client models are in `client.Models` namespace
- API models are in `StudyTracker.Api.Models` namespace
- Services follow the same name on client and API (e.g., `StudySessionService`)
- Constitution at `.specify/memory/constitution.md` defines non-negotiable principles

## SpecKit Workflow

Use `/speckit-*` commands for spec-driven development:
1. `/speckit-constitution` — Project principles
2. `/speckit-specify` — Feature specification
3. `/speckit-plan` — Implementation plan
4. `/speckit-tasks` — Task breakdown
5. `/speckit-implement` — Execute tasks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LanreAdetola)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LanreAdetola)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
