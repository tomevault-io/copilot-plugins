## statusapp

> This document provides guidance for agents that operate on StatusApp. It captures the repository layout, conventions, core data flows, and essential commands needed to build, test, run, and extend the project.

# AGENTS

This document provides guidance for agents that operate on StatusApp. It captures the repository layout, conventions, core data flows, and essential commands needed to build, test, run, and extend the project.

## Quick Context
StatusApp is a Go-based terminal UI (Bubble Tea) app that aggregates several domain data sources (Tailscale, Weather, Schedule, TrueNAS, UpCloud, HostHatch, Exaroton, etc.) and renders a real-time dashboard. Code is organized in domain-specific internal packages (internal/tailscale, internal/weather, internal/schedule, internal/truenas, internal/upcloud, internal/hosthatch, internal/exaroton, internal/syncthing, internal/clock, internal/common). The UI wiring is in cmd/* (main.go, view.go, fetch.go, update.go).

## What an Agent Should Know
- Architecture
  - Domain-driven structuring: each internal/<domain> package contains its own client.go (API interactions), types.go (models), and view.go (UI rendering). This keeps data fetching, processing, and presentation co-located. See internal/tailscale/, internal/weather/, internal/schedule/ as exemplars. file references: internal/tailscale/client.go:12-16, internal/weather/types.go:5-12, internal/schedule/view.go:11-18
  - Client pattern: a Client struct with a NewClient constructor encapsulates HTTP details; methods implement domain-specific data fetches. Interfaces (e.g., MachineGetter in tailscale) enable testability through mocks. See internal/tailscale/client.go:27-38 and internal/tailscale/types.go:5-16.
  - Rendering: View functions take domain data and return strings for Bubble Tea rendering; they are domain-local (e.g., internal/weather/view.go, internal/schedule/view.go). See internal/weather/view.go:15-23 and cmd/view.go:52-60.

- Key Commands & Build Workflow
  - Build scripts: scripts/Build.sh builds the CLI into deployments/StatusApp (go build -o ../deployments/StatusApp from cmd/). Use this for CI-like packaging. See scripts/Build.sh:3-5. 
  - Tests: Go tests run with go test ./...; per-package tests exist (e.g., internal/schedule/client_test.go). See internal/schedule/client_test.go:1-9 and docs/codebase.md for testing direction.
  - Environment: Run with a .env file; main.go loads env via godotenv; Weather/Tailscale/Auth rely on env vars. See cmd/main.go:141-149 and README example of environment variables in README.md: sections about WEATHER_PROVIDER and .env usage.

- Data Flow (What Happens When)
  - Data sources: domain packages implement their own client logic; Data is consumed by View logic, which is orchestrated by cmd/view.go. Examples: tailscale View feed into main screen; weather and clock render together. See cmd/view.go:12-21, 64-75; internal/tailscale/view.go: lines unknown here, but present in repo.
  - UI wiring: main loop in cmd/main.go starts Bubble Tea program and delegates rendering to domain view helpers via newModel(). See cmd/main.go:25-33 and cmd/view.go:79-87.

- Testing Approach
  - Domain tests exist for many packages (client_test.go, view_test.go). Tests rely on mock data and httptest servers to validate API logic without real network calls. See internal/schedule/loader.go (testability via file path param) and docs/ProjectUpdate.md describing tests patterns (historical reference).

- Gotchas & Non-Obvious Things
  - Package layout uses internal/ for private app code; external sharing is limited by Go's internal rule. Follow the existing domain structure rather than central models/renderers; see Restructure.md recommendations. internal/ is domain-scoped; avoid cross-domain coupling.
  - The UI uses a number of styling constants in configs/constants.go (colors, styles, width). Changes here ripple through all domain views. See configs/constants.go: defines ScheduleStyle, ClockStyle, etc.
  - The project relies on environment-driven configuration (WEATHER_PROVIDER, WEATHER_ICON_PATH, etc.). Ensure correct env values are provided; the app loads .env at startup. See cmd/main.go for env loading and README for example keys (WEATHER_PROVIDER, WEATHER_ICON_PATH, etc.).

- Representative Files to Inspect (Patterns & Best Places to Start)
  - Root and config: go.mod, configs/constants.go, README.md. See go.mod imports Bubble Tea, Lipgloss, etc. and constants.go for UI constants.
  - Domain clients and models: internal/tailscale/client.go (Client, NewClient, GetMachines, GetKeyExpiry); internal/tailscale/types.go (Device, Devices, Key). See internal/tailscale/client.go:27-38 and internal/tailscale/types.go:5-16.
  - Weather domain: internal/weather/client.go (GetCurrentWeather, GetWaterTemperature); internal/weather/types.go; internal/weather/view.go. See internal/weather/client.go:62-70 and internal/weather/types.go:35-41.
  - Schedule domain: internal/schedule/client.go (LoadSchedule); internal/schedule/types.go; internal/schedule/view.go. See internal/schedule/client.go:12-13 and internal/schedule/types.go:5-12.
  - UI wiring: cmd/main.go, cmd/view.go. See cmd/main.go:29-46 and cmd/view.go:79-87.
 
## How to Extend or Modify
- Follow the domain-driven pattern: keep data types with their owning domain, provide a Client with a constructor, and a View for rendering within the same domain package.
- Add tests alongside new code (client_test.go, view_test.go) and use httptest for HTTP layers. See docs/ProjectUpdate.md for high-level guidance on testing overhaul.
- Use go test ./... to validate changes; update go.mod via go mod tidy if new dependencies are added.

## Quick Start Checklist
- [ ] Inspect root files: go.mod, configs/constants.go, README.md
- [ ] Review internal packages: tailscale, weather, schedule, truenas, upcloud, hosthatch, exaroton, syncthing, clock, common
- [ ] Review cmd entrypoints: main.go, view.go
- [ ] Try a local build: scripts/Build.sh or go build ./...
- [ ] Run tests: go test ./...
- [ ] Provide or load a .env appropriate for your environment

## Notable File References
- internal/tailscale/client.go:12-16, 27-38, 41-73
- internal/tailscale/types.go:5-16
- internal/weather/client.go:14-18, 62-70, 189-226
- internal/weather/view.go:11-18, 25-33, 35-44
- internal/schedule/client.go:12-16, 28-36, 68-74
- internal/schedule/view.go:11-18, 82-96
- cmd/main.go:19-27, 39-47, 139-149
- configs/constants.go:5-10, 30-44
- README.md:3-9, 31-47, 81-99
- docs/Restructure.md:39-62

---
> Source: [manfredbjorlin/StatusApp](https://github.com/manfredbjorlin/StatusApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
