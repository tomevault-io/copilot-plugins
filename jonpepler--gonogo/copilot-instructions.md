## gonogo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Vision

**gonogo** is a mission control SPA for Kerbal Space Program. It operates in two modes within the same app:

- **Main screen** (`/`) — connects to KSP data sources, hosts a live telemetry dashboard, and distributes data to connected station screens via PeerJS.
- **Station screen** (`/station`) — a peer-connected dashboard whose layout and role are stored in `localStorage`. Stations can optionally pull a saved config from the main screen. There are no per-station routes; each device at `/station` is its own independent station.

The defining feature is a **context-aware, extensible dashboard component system**: components self-register into a global registry, and the dashboard orchestrator renders whatever is registered. External packages (not in this repo) can add components and themes using the same API as the built-in library.

---

## Monorepo Structure

```
packages/
  core/       — Plugin registry, shared TS types, React contexts, GO/NO-GO system
  components/ — Built-in dashboard component library (uses core registry)
  app/        — Vite + React SPA (main screen + station mode)
  proxy/      — Lightweight Fastify server: kOS telnet → WebSocket bridge
```

**Tooling:** pnpm workspaces + Turborepo. Package names use the `@gonogo/` scope.

---

## Commands

```bash
pnpm install          # install all workspace dependencies
pnpm dev              # run app (Vite) and proxy server in parallel
pnpm build            # build all packages via Turborepo
pnpm test             # run tests across all packages
pnpm lint             # lint all packages
pnpm --filter @gonogo/app dev       # run only the SPA
pnpm --filter @gonogo/proxy dev     # run only the proxy server
pnpm --filter @gonogo/core test     # test a single package
```

---

## Architecture

### Data Flow

```
KSP (Telemachus Reborn HTTP/WS) ──→ Main screen (direct, React Query)
KSP (kOS via telnet)            ──→ @gonogo/proxy (Fastify + telnet client)
                                         └──→ Main screen (WebSocket)
Main screen ←──→ Station screens (PeerJS data channels, via peerjs.com broker)
```

Telemachus Reborn is a standard HTTP/WebSocket API — the browser talks to it directly. The proxy server is **only required for kOS integration**; without it, all other features still work. The app must display proxy connection status prominently in the UI.

### `@gonogo/core`

The foundation for everything extensible:

- **Plugin registry** — `registerComponent(def)`, `registerTheme(def)`, and `registerDataSource(def)` are the three extension points. Calling these at module load time is all that's needed to extend the app.
- **Shared TypeScript types** — `ComponentDefinition`, `ThemeDefinition`, `DataSourceDefinition`, `StationConfig`, `DataRequirement`, `Behavior`, etc.
- **React contexts** — `DashboardContext` (current layout, orchestrator state), `PeerContext` (PeerJS connection state), `StationContext` (station identity/role from localStorage).
- **GO/NO-GO system** — aggregates GO/NO-GO state across all active stations. A component can declare `behaviors: ['gonogo-participant']` in its definition to contribute to the global state.
- **Data source interface (repository pattern)** — all data sources implement a common `DataSource` interface:
  ```ts
  interface DataSource {
    id: string;
    name: string;
    connect(): Promise<void>;
    disconnect(): void;
    status: DataSourceStatus; // 'connected' | 'disconnected' | 'error'
    schema(): DataKey[];                                      // available keys
    subscribe(key: string, cb: (value: unknown) => void): () => void;
  }
  ```
  The orchestrator resolves component `dataRequirements` against registered sources at runtime. Components declare keys using a normalised format (e.g. `'vessel.altitude'`); each `DataSource` implementation maps its own API response shape onto those keys. This means components are data-source-agnostic.

### `@gonogo/components`

The built-in component library. Each component file calls `registerComponent()` on import — there is no central index that manually lists them; the orchestrator just needs to import the package and registration happens automatically.

Components declare their `dataRequirements` (e.g. `['vessel.altitude']`) so the orchestrator knows what data to subscribe to. The data layer resolves requirements against registered data sources.

Components are styled with **styled-components**. Component names and styled sub-components follow BEM-inspired naming for readability (e.g. `AltitudeGauge`, `AltitudeGauge__Label`, `AltitudeGauge__Value`).

### `@gonogo/app`

The Vite SPA. Key responsibilities:

- **Dashboard orchestrator** — a layout engine built on [React Grid Layout](https://github.com/react-grid-layout/react-grid-layout) (`ResponsiveGridLayout`) that reads the current layout config and renders registered components by ID. It does not hardcode any component — it only knows about the registry. Positions are stored in **grid units** (column/row spans), not pixels, so layouts are resolution-independent. The serialised layout format stores a per-breakpoint map (`lg`, `md`, `sm`, etc.) so the grid reflows across screen sizes. Per-instance component config is stored alongside the layout.
- **Telemachus Reborn client** — direct HTTP/WS integration using React Query. Components that need telemetry data declare requirements; the orchestrator resolves and subscribes to the right endpoint.
- **kOS WebSocket client** — connects to `@gonogo/proxy`. The proxy status is shown persistently in the main screen UI. If the proxy is not reachable, affected components degrade gracefully.
- **PeerJS integration** — the main screen acts as the peer host. Stations connect as peers. The main screen distributes a serialised snapshot of data to all peers; stations can also send state back (e.g. GO/NO-GO votes).
- **Station config** — localStorage-first. Stations can request a config from the main screen over PeerJS; the main screen can push saved configs to connecting stations.

### `@gonogo/proxy`

A minimal Fastify server. Its only job is bridging kOS telnet sessions to a WebSocket that the browser can consume. It should be runnable with a single command and have clear setup instructions in its own README. The main screen should show an unambiguous status indicator for this connection.

---

## Extension Pattern

Both components and themes follow the same self-registration pattern:

```ts
// An external npm package can do this:
import { registerComponent } from '@gonogo/core';

registerComponent({
  id: 'my-custom-gauge',
  name: 'My Custom Gauge',
  category: 'telemetry',
  component: MyCustomGauge,
  dataRequirements: ['vessel.altitude'],
  behaviors: [],           // e.g. ['gonogo-participant'] to join GO/NO-GO
  defaultConfig: {},
});
```

```ts
import { registerTheme } from '@gonogo/core';

registerTheme({
  id: 'retro-nasa',
  name: 'Retro NASA',
  theme: { colors: { ... }, fonts: { ... } }, // passed to styled-components ThemeProvider
});
```

The built-in `@gonogo/components` package models this pattern exactly — it is not treated as special by the orchestrator.

---

## Testing Philosophy

Prefer tests that mock as little of the system as possible. Use [Mock Service Worker (MSW)](https://mswjs.io/) to intercept at the network boundary rather than mocking modules.

- **Integration tests** (in `@gonogo/app`) use MSW WebSocket/HTTP handlers to simulate KSP APIs. The real data source, real hook, and real component all run — only the network is intercepted. This is the preferred form for tests involving connection status or data flow.
- **Unit tests** (in `@gonogo/core`, `@gonogo/components`) use the real registry with simple disconnected fixture data sources. No `vi.mock()` of internal modules. MSW is only needed when a test actually triggers a network call.
- Avoid mocking `useDataSources` or other core hooks in component tests — render the real component with real registry state instead.
- **`act()` warnings are always our bug** — never dismiss them. They mean a state update is escaping the `act()` boundary. The fix is usually to make the async function resolve *after* the state update (e.g. `connect()` should resolve inside the `open` handler, not before it fires).

---

## Key Design Constraints

- **Main screen is the sole KSP data consumer.** Stations never talk to KSP directly; they receive data exclusively from the main screen over PeerJS.
- **Proxy is optional infrastructure, not a core dependency.** The app must function (minus kOS features) without it. Never make the proxy a hard startup requirement.
- **PeerJS broker is configurable.** Default to `0.peerjs.com` but expose a config option (environment variable or settings UI) to point at a self-hosted broker.
- **Themes are runtime-switchable.** The ThemeProvider must be driven by the active theme from the registry, not hardcoded at build time.
- **Station identity is localStorage-first.** Never assume a station has a server-side identity. Server-saved configs are a convenience layer on top of a fully local-first station.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonpepler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
