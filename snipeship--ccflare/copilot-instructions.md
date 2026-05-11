## ccflare

> Guidance for code agents working in this repository.

# AGENTS.md

Guidance for code agents working in this repository.

This file is the repo-local source of truth for:

- workspace structure
- package ownership
- commands to run before handing work back
- DRY / refactor expectations
- where to look in `docs/` before making architectural changes

## Engineering Standard

Always optimize for code that is:

- clear to read
- easy to maintain
- appropriately DRY
- locally coherent
- boring in the good sense

Prefer small, obvious abstractions over clever ones. Remove repetition when it
reduces maintenance cost, but do not introduce generic helper layers that make
the code harder to follow than the duplication they replace.

The target is not “maximum abstraction.” The target is code that a strong
engineer can understand and modify quickly without guessing at hidden behavior.

## What This Repo Is

`ccflare` is a Bun/TypeScript monorepo for a multi-provider AI proxy.

Core product behavior:

- provider-native HTTP passthrough for `anthropic`, `openai`, `claude-code`, and `codex`
- WebSocket proxying for supported upstreams
- account failover and session-based selection
- SQLite-backed request/account/auth-session persistence
- management API
- browser dashboard
- terminal UI

High-level docs:

- [`README.md`](README.md)
- [`docs/index.md`](docs/index.md)
- [`docs/architecture.md`](docs/architecture.md)
- [`docs/data-flow.md`](docs/data-flow.md)

## Reading Order

If you are changing behavior, read in this order:

1. [`README.md`](README.md)
2. [`docs/architecture.md`](docs/architecture.md)
3. the domain-specific doc for your area

Use these targeted references:

- API/control plane: [`docs/api-http.md`](docs/api-http.md)
- runtime/request flow: [`docs/data-flow.md`](docs/data-flow.md)
- provider behavior: [`docs/providers.md`](docs/providers.md)
- config rules: [`docs/configuration.md`](docs/configuration.md)
- database/persistence: [`docs/database.md`](docs/database.md)
- load-balancing/session strategy: [`docs/load-balancing.md`](docs/load-balancing.md)
- TUI behavior: [`docs/tui.md`](docs/tui.md)
- deployment/paths/build outputs: [`docs/deployment.md`](docs/deployment.md)
- contribution expectations: [`docs/contributing.md`](docs/contributing.md)

## Current Workspace Layout

```text
ccflare/
├── apps/
│   ├── desktop/         # Desktop shell
│   ├── lander/          # Landing page
│   ├── server/          # Main Bun server entrypoint
│   ├── tui/             # Terminal UI + local TUI core
│   └── web/             # Browser dashboard
├── packages/
│   ├── api/             # REST/SSE handler layer and API transport types
│   ├── config/          # Config loading/defaults/env/path handling
│   ├── core/            # Constants, validation, lifecycle, DI, pricing
│   ├── database/        # SQLite schema, repositories, async writer
│   ├── http/            # Shared HTTP client/response/header/error utilities
│   ├── logger/          # Logging, log bus, file writer
│   ├── oauth-flow/      # Shared OAuth onboarding flow
│   ├── providers/       # Provider registry + provider implementations
│   ├── proxy/           # HTTP/WebSocket proxy path
│   ├── runtime-server/  # Server bootstrap/orchestration
│   ├── types/           # Shared domain and transport types
│   └── ui/              # Shared UI presenters/components/constants
└── docs/
```

## Package Ownership

Use these ownership rules when deciding where code belongs.

### `apps/server`

Owns:

- the top-level server entrypoint
- thin wrapper around `@ccflare/runtime-server`

Do not move orchestration logic here if it can live in `runtime-server`.

### `apps/tui`

Owns:

- the TUI app
- TUI-only core wrappers under `apps/tui/src/core`
- screen components and terminal interaction behavior

TUI-local logic should stay here unless it is reused by another app.

### `apps/web`

Owns:

- the browser dashboard
- dashboard-only chart composition and page models
- static web build output

Shared UI display logic belongs in `@ccflare/ui`, not here.

### `packages/runtime-server`

Owns:

- runtime bootstrap
- Bun server startup
- request routing between `api`, `proxy`, and `web`
- startup maintenance
- graceful shutdown

This package is intentionally orchestration-heavy. Do not turn it into a grab-bag for provider or API logic.

### `packages/api`

Owns:

- management/control-plane handlers
- `APIRouter`
- API transport response types specific to the HTTP API surface

Management endpoint behavior belongs here.

### `packages/proxy`

Owns:

- provider-path request forwarding
- websocket proxy handling
- retry/failover behavior
- session strategy implementation
- request event emission
- worker handoff for usage/payload processing

Data-plane request forwarding belongs here.

### `packages/providers`

Owns:

- provider-specific URL construction
- auth header preparation
- OAuth provider adapters
- token refresh behavior
- provider-native rate-limit parsing
- provider-specific usage parsing

Avoid moving app/session orchestration into provider packages.

### `packages/oauth-flow`

Owns:

- shared OAuth onboarding flow for both API and TUI
- auth session state orchestration
- PKCE setup and completion flow

Keep it separate while both `@ccflare/api` and `ccflare` use it.

### `packages/database`

Owns:

- schema
- migrations
- repositories
- async DB write path
- database factory / facade

Persistence logic should move here before it leaks into runtime or proxy packages.

### `packages/http`

Owns:

- HTTP client
- JSON/SSE response helpers
- header sanitization
- shared HTTP error model

### `packages/core`

Owns:

- constants
- validation
- lifecycle/disposables
- pricing helpers
- DI container + service keys

### `packages/types`

Owns:

- canonical domain types
- shared transport types
- guards and small validation helpers

### `packages/ui`

Owns:

- shared presenters
- shared display helpers
- shared UI constants/theme data
- shared parser/display components that are UI-facing but framework-neutral

## Naming Conventions

- Apps use short names: `server`, `tui`, `web`
- Shared packages use `@ccflare/*`
- Prefer specific names over vague names:
  - `api`, not `http-api`
  - `http`, not `http-common`
  - `ui`, not `ui-common`
- Keep `runtime-server` as-is; `runtime` would be too vague

## Import Conventions

- Use workspace imports for shared packages: `@ccflare/*`
- Use local relative imports inside an app’s internal modules
- Do not import from `dist/`
- Do not import from app folders into packages
- Keep imports aligned with actual ownership boundaries

Before adding a new package, ask:

1. Does this have 2+ real consumers?
2. Does it have a stable API boundary?
3. Does it have materially different runtime/build/dependency needs?

If not, prefer a local module over a workspace package.

## Commands

### Main app commands

```bash
bun run ccflare
bun run server
bun run start
bun run dev:server
bun run dev:dashboard
```

### Build and quality

```bash
bun run build
bun run build:dashboard
bun run build:tui
bun run build:server
bun run build:desktop
bun run build:lander

bun run test
bun run typecheck
bun run lint:check
bun run verify
bun run format
bun run lint
```

### Duplicate detection

Production-code duplication is tracked with `jscpd`:

```bash
bun run dupes
```

Config file:

- [`.jscpd.json`](.jscpd.json)

Current intent:

- measure production/runtime duplication
- ignore tests, test helpers, generated output, and the static lander HTML

Do not “game” this metric with pointless abstractions. Only refactor when the result is clearer or materially more maintainable.

## Required Checks Before Hand-Off

For normal code changes:

```bash
bun run verify
```

For docs-only changes:

- ensure references and paths are still correct
- grep for stale package names if the change followed a package refactor

For targeted package-only changes, run the narrowest meaningful checks first, then expand if needed.

## Refactor Policy

Prioritize:

- duplication in production/runtime code
- repeated provider/runtime orchestration
- repeated chart wrappers and parser logic
- stale package boundaries

Be cautious with:

- same-file JSX repetition that may become harder to read if over-abstracted
- tiny helpers that add indirection without reducing maintenance cost

Good DRY targets:

- repeated proxy/runtime workflow helpers
- provider-specific refresh/rate-limit/usage patterns
- shared chart shells
- repeated parser block-handling logic

Bad DRY targets:

- collapsing unlike concepts into generic helper bags
- introducing “framework” layers just to satisfy a metric
- moving app-local logic into a shared package without a second consumer

## Docs Policy

If you change package names, ownership, commands, build paths, or architecture, update:

- [`README.md`](README.md)
- relevant files in [`docs/`](docs/)

At minimum, check:

- [`docs/index.md`](docs/index.md)
- [`docs/architecture.md`](docs/architecture.md)
- [`docs/data-flow.md`](docs/data-flow.md)
- [`docs/contributing.md`](docs/contributing.md)
- [`docs/deployment.md`](docs/deployment.md)

## TUI Notes

When changing `apps/tui`:

- keep screen logic in app-local modules unless reused elsewhere
- preserve keyboard behavior and navigation expectations
- prefer readability over clever abstractions in screen components
- use `@ccflare/ui` for shared display helpers where appropriate

## Web Notes

When changing `apps/web`:

- prefer shared chart shells over repeating `recharts` frame code
- keep page-model/data-fetch orchestration in hooks
- keep presentational/shared display logic in `@ccflare/ui`

## Provider Notes

When changing `packages/providers`:

- keep protocol-specific behavior in provider implementations
- preserve provider-native payloads
- keep refresh/rate-limit parsing provider-specific unless a helper is genuinely shared
- avoid coupling provider modules to app-layer onboarding flow

## Proxy Notes

When changing `packages/proxy`:

- keep HTTP and WebSocket routing behavior aligned
- prefer shared request/session helpers over copy/pasted flow logic
- preserve event emission semantics and worker message contracts

## Database Notes

When changing `packages/database`:

- prefer repository/facade updates over call-site SQL
- preserve migration idempotence
- keep write-path changes compatible with async writer behavior

## If You’re Unsure

Read:

1. [`docs/architecture.md`](docs/architecture.md)
2. [`docs/data-flow.md`](docs/data-flow.md)
3. the package you’re about to edit

Then choose the smallest boundary that keeps ownership clear.

---
> Source: [snipeship/ccflare](https://github.com/snipeship/ccflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
