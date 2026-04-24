## lantern

> Lantern is a real-time SvelteKit control panel for homelab/service monitoring. It follows a TypeScript-first architecture:

1. System Overview

Lantern is a real-time SvelteKit control panel for homelab/service monitoring. It follows a TypeScript-first architecture:

Canonical policy filename is `AGENTS.md` (uppercase). Do not create a second lowercase variant.

    Core (SvelteKit): UI routes, API endpoints, editor UX, and server/runtime orchestration.

    Connectors (Node/TS): Service-specific integrations under `src/lib/server/connectors` fetch and normalize external data.

    State Management: Persistent config lives in `config/widgets.json`; runtime widget data is cached in memory and streamed over SSE.

2. Path,Role,Codex Instruction
/src/routes/+page.svelte,Lantern View,"Main runtime Lantern UI. Keep display concerns here; avoid server fetch logic in component internals when route/server modules already provide it."

/src/routes/editor/+page.svelte,Visual Editor,"Editor-only authoring and settings UI. Keep widget editing flows here, not in the runtime Lantern page."

/src/routes/api/*/+server.ts,HTTP API Handlers,"All endpoint logic lives in route-scoped `+server.ts` files grouped by API domain (`config`, `widgets`, `stream`, `monitor-test`, `health-test`, `background`)."

/src/lib/server/state.ts,Runtime State Engine,"Owns config normalization, polling cadence, runtime cache, and refresh lifecycle. Keep scheduling/state orchestration centralized here."

/src/lib/server/stream.ts,SSE Broadcast Layer,"SSE client registration, heartbeat, and event fanout helpers only."

/src/lib/server/security.ts,Node/Security Helpers,"Node parsing/allowlisting/security helpers used by routes/connectors."

/src/hooks.server.ts,Global Request Hooks,"Optional global request guards (auth/rate limiting). Keep behavior non-blocking by default unless explicitly enabled."

/src/lib/server/connectors/*.ts,Service Connectors,"External service adapters (Komodo, Technitium, ARR stack, etc.). Keep provider-specific I/O and normalization here."

/src/lib/server/connectors/http.ts,HTTP Utility,"Shared request/retry/timeout helper for connectors. Reuse this instead of ad-hoc fetch wrappers."

/src/lib/widgets/types.ts,Widget Schema,"Shared widget/domain types and config contracts only."

/src/lib/widgets/registry.ts,Widget Registry,"Widget kind/source registration and metadata wiring."

/src/lib/components/widgets/*.svelte,Widget Renderers,"Per-widget presentation components. Keep them focused on rendering and local interaction."

/src/lib/components/*.svelte,Shared UI Components,"Reusable shell/editor/runtime components (cards, drawers, renderer plumbing)."

/src/lib/stores/widgets.ts,Client Widget Store,"Client-side widget state and subscriptions."

/src/lib/styles/global.css,Global Styles,"Global layout/theme/style primitives. Avoid dumping feature-specific one-off styles without clear sectioning."

/src/lib/shared/*.ts,Shared Runtime/Editor Utilities,"Cross-route helpers used by both Lantern and editor (tabs, widget catalog, future shared settings/layout helpers)."

/config/widgets.json,Config Ledger,"Primary persisted Lantern/widget configuration; treat as source-of-truth config data."

/static/uploads/*,Uploaded Assets,"User-uploaded backgrounds/assets persisted across rebuilds."

3. Workflow Rules

- Avoid restarting/recreating Docker services for minor changes. Use type checks/build checks first. Rebuild/restart containers only for major runtime changes or when explicitly requested.
- After completing requested changes, always run `git add .` and create a commit with a concise summary message.
- For feature work that touches route/server structure, run a hotspot check before finishing:
  - `wc -l src/routes/**/*.ts src/routes/**/*.svelte src/lib/server/**/*.ts 2>/dev/null | sort -nr | head -n 20`
- Auth/security posture for now: do not require authentication by default. Any auth gate must be optional and explicitly enabled via environment/config flags.

4. Structure Guardrails (Important)

- New HTTP endpoints must be added as `src/routes/api/<domain>/+server.ts` (or an existing API domain file), not embedded in page components.
- Keep route/page files thin: heavy server-side orchestration belongs in `src/lib/server/*`.
- New service integrations must live in `src/lib/server/connectors/` and be wired through shared state/route layers, not directly from UI components.
- Reuse shared helpers before adding new ones:
  - `src/lib/server/connectors/http.ts` for HTTP calls/retry/timeout behavior.
  - `src/lib/server/state.ts` for runtime polling/refresh scheduling.
  - `src/lib/server/stream.ts` for SSE client/event handling.
  - `src/lib/widgets/types.ts` and `src/lib/widgets/registry.ts` for widget contracts/registration.
- Keep library modules import-safe: no long-running loops or external side effects at import time.
- When adding a widget kind/source, update all required touchpoints in the same change:
  - `src/lib/widgets/types.ts`
  - `src/lib/widgets/registry.ts`
  - `src/lib/components/WidgetRenderer.svelte`
  - relevant `src/lib/components/widgets/*.svelte`
  - editor controls in `src/routes/editor/+page.svelte` (if configurable)
- Keep Lantern/editor UI modular: reusable controls go in `src/lib/components/*`; widget-specific markup stays in `src/lib/components/widgets/*`.
- Keep styles centralized and organized in `src/lib/styles/`; avoid creating an unstructured single-file style dump.
- Prefer adding focused files/modules when a touched file is already large; avoid re-creating monolithic “god files.”
- Soft limit: if a TS/Svelte file is already ~700+ LOC and a change adds a new concern, split to a new focused module instead of extending it.
- Prioritize these next splits when touching related code:
  - Move duplicated Lantern/editor settings defaults + normalization into `src/lib/shared/dashboardSettings.ts`.
  - Move execution-node parsing/selection into `src/lib/shared/executionNodes.ts`.
  - Move shared color/number/css helpers into `src/lib/shared/styleUtils.ts`.
  - Break `src/routes/editor/+page.svelte` into focused editor modules/components (tabs, typography, add-widget wizard, monitor node controls).
  - Break `src/lib/server/state.ts` polling/refresh logic into domain modules under `src/lib/server/state/`.
- Add/update tests for behavior changes when test coverage exists or is introduced; keep validation compatible with project scripts.
- If moving files between routes/lib/components/connectors, update all callsites (imports, README, Docker/runtime assumptions) in the same change.
- Prefer ESM module imports (`$lib/...`, relative TS module imports) over duplicated/local copy-paste logic.

5. Validation + Handoff

- Prefer `npm run check` for final validation (Svelte/TypeScript checks).
- For route/server or runtime behavior changes, also run `npm run build`.
- If a change affects runtime commands or deployment assumptions, verify both route paths and container/runtime invocation paths.
- End each completed update by staging all changes and committing with a clear summary:
  - `git add .`
  - `git commit -m "<concise summary>"`

---
> Source: [bobbydd021-code/Lantern](https://github.com/bobbydd021-code/Lantern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
