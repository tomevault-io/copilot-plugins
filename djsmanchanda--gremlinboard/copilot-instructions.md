## gremlinboard

> GremlinBoard is a modular live widget board with AI-assisted widget creation, background microservices, and a strict grid layout system.

# AGENTS.md — GremlinBoard

GremlinBoard is a modular live widget board with AI-assisted widget creation, background microservices, and a strict grid layout system.

## Product Shape

GremlinBoard has three layers:
1. Board UI
2. Widget runtime
3. Spec / generation tooling

The board is the surface.
Widgets are the units.
Microservices are the runtime.

## Stack

Frontend:
- Next.js App Router
- TypeScript
- React
- Tailwind CSS

Backend:
- FastAPI
- Python 3.12+
- Pydantic
- Async-first services

Desktop wrapper:
- Tauri, only if/when desktop packaging is needed

## Core Rules

- Use a strict grid.
- The board may scale from 4 to 8 columns based on available width.
- Allowed widget sizes only:
	- 1x1
	- 1x2
	- 2x2
	- 4x2
	- 2x4
	- 4x4
- Do not introduce arbitrary widget sizes.
- Every widget must have a manifest.
- Every widget must be registered before use.
- Every widget service must be lightweight and disposable.

## Widget Contract

Every widget must define:
- id
- name
- description
- min_size
- preferred_size
- refresh_policy
- lifecycle_state
- permissions
- data schema
- renderer target

Widgets must never bypass the registry.

## Microservice Contract

Every microservice must support:
- start()
- stop()
- health()
- get_state()

Prefer:
- async code
- event-driven updates
- minimal polling
- graceful shutdown

## AI / Spec Rules

The spec widget is the only approved entry point for AI-generated widget creation.

AI output must be staged in this order:
1. spec draft
2. validation
3. scaffold
4. codegen
5. review
6. install

Do not allow direct deploy from raw AI output.

## Runtime Rules

The runtime must manage:
- widget lifecycle
- refresh intervals
- health checks
- state persistence
- event delivery
- cleanup on removal

## UI Rules

The board should feel like a live control surface.

Required behaviors:
- default to locked View mode for monitoring
- use Edit mode for drag, resize, widget controls, and source settings
- drag from the top interaction band, excluding resize corners
- resize from the bottom-right corner by allowed ratios only
- show dashed resize previews for allowed sizes that fit the current column count, with the nearest target highlighted
- snap to grid
- persist layout
- persist board appearance settings such as density presets
- expose freshness, uptime, mode, and restart stats only through the widget Stats toggle
- surface widget alerts in three visible categories: red `critical` for broken behavior, yellow `alert` for non-fatal issues, and explicit green `completed` for successful task completion
- keep permanent widget chrome minimal so widgets can show more data
- support quick add via command box

## Quality Rules

Before merging any major change:
- typecheck
- lint
- build
- runtime sanity check
- widget registry check

## Developer Mindset

Prefer:
- small modules
- clear boundaries
- stable interfaces
- predictable data flow

Avoid:
- giant components
- hidden coupling
- hardcoded widget logic
- one-off hacks

---
> Source: [djsmanchanda/GremlinBoard](https://github.com/djsmanchanda/GremlinBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
