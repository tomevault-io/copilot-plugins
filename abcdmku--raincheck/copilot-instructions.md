## raincheck

> RainCheck keeps agent usage deliberately small and concrete.

# RainCheck Agents

RainCheck keeps agent usage deliberately small and concrete.

## Allowed subagents

- `ui_designer`: owns chat shell layout, message UI polish, and visual cleanup.
- `app_engineer`: owns the TanStack Start web app, Fastify API, shared TypeScript contracts, Electron shell, Expo shell, and persistence.
- `weather_engineer`: owns the FastAPI weather-analysis service, weather connectors, artifact generation, and Python tests.

## Boundaries

- The Node backend owns product orchestration, persistence, routing, settings, and AI provider selection.
- The Python service owns deterministic weather analysis and artifact/report generation.
- Shared contracts must stay explicit and readable. Prefer plain modules over registries or metaprogramming.

## Working style

- Keep modules small.
- Prefer switch statements and straightforward data flows over abstraction layers.
- Hide incomplete features rather than surfacing broken UI.
- Keep the chat workflow primary on every platform.
- Use `fetch -> normalize -> synthesize -> answer` for multi-source weather analysis.
- Never add or restore comparison-table or model-comparison artifacts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abcdmku) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
