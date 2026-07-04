## openscience

> This file is the working guide for AI agents and human contributors inside this repository. Follow it together with [CONTRIBUTING.md](CONTRIBUTING.md) and [CONTRIBUTING.zh.md](CONTRIBUTING.zh.md).

# DeepOrganiser Agent Guide

This file is the working guide for AI agents and human contributors inside this repository. Follow it together with [CONTRIBUTING.md](CONTRIBUTING.md) and [CONTRIBUTING.zh.md](CONTRIBUTING.zh.md).

## Project Core

DeepOrganiser is an Electron + React desktop app and standalone WebUI shell around a bundled Rust backend binary named `deeporganiser-core`.

The important mental model:

- The TypeScript/Electron code owns the app shell, renderer UI, packaging, WebUI static server, Electron/Web bridge, config client, type contracts, migrations from legacy stores, bundled MCP scripts, and local process lifecycle.
- `deeporganiser-core` owns most runtime business APIs: auth, users, providers, conversations, messages, MCP, cron, agents, remote agents, teams, assistants, settings, and realtime events.
- The renderer almost never talks to Electron main directly for business data. It calls the backend through `ipcBridge`, which is now mostly HTTP/WebSocket wrappers over `/api/*` and `/ws`.
- Electron desktop and standalone WebUI share the same renderer bundle, but differ in transport and auth. Desktop injects a backend port through preload. WebUI uses same-origin `/api/*`, `/login`, `/logout`, and `/ws` proxied by `@deeporganiser/web-host`.

When something is not implemented in TypeScript, check whether it is an `deeporganiser-core` API surface before adding duplicate frontend/main-process logic.

## Repository Map

Top-level layout:

- `packages/desktop/`: Electron app, renderer, preload, main process utilities, shared TS contracts.
- `packages/web-host/`: starts or attaches to `deeporganiser-core`, serves `out/renderer`, proxies `/api/*`, `/login`, `/logout`, `/ws`, and `/api/stt/stream`.
- `packages/web-cli/`: CLI helpers for browser opening and WebUI admin password flow.
- `packages/shared-scripts/`: shared release/setup helpers, including bundled `deeporganiser-core` preparation/verification.
- `scripts/`: build, packaging, WebUI launch, migrations, debug, i18n, and release utilities.
- `resources/bundled-deeporganiser-core/<platform>-<arch>/`: expected location for the `deeporganiser-core` binary in dev/package flows.
  This directory is gitignored and usually absent in a fresh worktree.
- `docs/contributing/`: contributor setup and structure rules. Some references in `docs/README.md` point to directories not present in this clone, so prefer source when docs disagree.
- `docs/prds/`: product requirements. Do not reorganize product-owned docs without explicit permission.
- `examples/`: extension examples, including ACP adapters, channels, assistants, skills, themes, and settings tabs.
- `.claude/skills/`: repository-local skills. Use them when their trigger conditions apply.

## Runtime Architecture

Desktop startup:

1. `packages/desktop/src/index.ts` runs first. It configures Chromium/app paths before any `app.getPath('userData')` call, enforces single-instance behavior, fixes GUI PATH, initializes Sentry/logging, starts storage, starts `deeporganiser-core`, creates the main window, and wires Electron lifecycle/tray/deep-link/WebUI behavior.
2. `packages/desktop/src/process/index.ts` initializes platform registration, storage, bridges, and main-process i18n.
3. `@deeporganiser/web-host` `BackendLifecycleManager` resolves `deeporganiser-core` via `packages/desktop/src/process/backend/binaryResolver.ts`, spawns it, waits for health/listening, and stores the backend port on `globalThis.__backendPort`.
4. `packages/desktop/src/preload/main.ts` exposes the Electron bridge and backend startup state to the renderer.
5. `packages/desktop/src/renderer/main.tsx` initializes config/i18n/theme, installs the browser bridge adapter, prefetches detected agents, and mounts the React app.

Standalone WebUI startup:

1. `scripts/webui.ts` optionally runs `bun run package`, resolves renderer assets, resolves `deeporganiser-core`, chooses a data dir and port, then calls `startWebHost`.
2. Default WebUI ports are dev `25809`, multi-instance dev `25810`, production `25808`.
3. Default standalone data dirs are `~/.openscience-web-dev`, `~/.openscience-web-dev-2`, or `~/.openscience-web`; they intentionally do not collide with Electron's default data dir.
4. `packages/web-host/src/static-server.ts` is a static server and reverse proxy only. Do not put business routes there.

Backend spawning:

- `packages/web-host/src/backend-launcher.ts` builds `deeporganiser-core` args: `--port`, `--data-dir`, optional `--parent-pid`, `--log-level`, `--app-version`, `--managed-resources-mode bundled`, `--log-dir`, `--work-dir`, and `--local`.
- It injects `DEEPORGANISER_CACHE_DIR`, `DEEPORGANISER_WORK_DIR`, and `DEEPORGANISER_LOG_DIR` so backend `/api/system/info` matches Electron-managed directories.
- It cleans registered child agent processes through `runtime/agent-process-registry.json` before backend startup.
- It skips Fetch-forbidden ports and validates startup with health checks.

## Local Backend Resources

`deeporganiser-core` is not built from this TypeScript repository during normal app startup. The desktop app expects a prebuilt backend bundle at:

```text
resources/bundled-deeporganiser-core/<process.platform>-<process.arch>/
- deeporganiser-core[.exe]
- manifest.json
- managed-resources/
```

Important workflow facts:

- `resources/bundled-deeporganiser-core` is intentionally ignored by git. A new `git worktree add ... HEAD` will not contain it.
- If the renderer shows "missing required local resources" or logs `DeepOrganiser Core startup failed while resolving backend binary`, first check this directory before changing frontend code.
- For the current macOS arm64 setup, the expected runtime key is `darwin-arm64`.
- The pinned backend version is `package.json#deepOrganiserCoreVersion`; in this checkout it is `v0.1.34`.

Ways to restore the bundle:

```bash
# Preferred when another local checkout already has a working bundle.
cp -a /path/to/working/DeepOrganiser-git/resources/bundled-deeporganiser-core resources/

# Or download/prepare the pinned backend bundle.
node scripts/prepareDeepOrganiserCore.js
```

Verify before launching Electron:

```bash
node - <<'NODE'
const { verifyBundledDeepOrganiserCoreResources } = require('./packages/shared-scripts/src/verify-bundled-deeporganiser-core-resources');
const result = verifyBundledDeepOrganiserCoreResources({
  resourcesDir: process.cwd() + '/resources',
  electronPlatformName: process.platform,
  targetArch: process.arch,
});
console.log(JSON.stringify(result, null, 2));
if (result.missing.length) process.exit(1);
NODE

resources/bundled-deeporganiser-core/darwin-arm64/deeporganiser-core --help
```

Desktop dev launches the backend with `--port 0`, so do not assume a fixed backend port. Read the actual port from logs such as `backendManager.start ready (port=...)`, `globalThis.__backendPort`, or the preload-injected `window.__backendPort`.

Backend startup triage:

- `Failed to start DeepOrganiser Core ... resolving backend binary`: missing binary or wrong `resources/bundled-deeporganiser-core/<runtimeKey>` layout.
- Installation integrity dialog about missing local resources: same root cause in packaged or package-like contexts.
- `fetch failed` immediately after launch: often means backend did not start, not that the renderer route is broken. Check the `deeporganiser-core` startup lines first.
- Runtime component failures after backend is healthy, especially around Node/ACP tools, usually relate to `managed-resources/` or the backend runtime preparation flow rather than `ipcBridge`.

## Cross-Process And API Boundaries

Hard boundaries:

| Layer         | Path                                                             | May use                                      | Must not use                                      |
| ------------- | ---------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------- |
| Electron main | `packages/desktop/src/index.ts`, `packages/desktop/src/process/` | Node.js, Electron main APIs, child processes | DOM APIs, React                                   |
| Preload       | `packages/desktop/src/preload/`                                  | `contextBridge`, `ipcRenderer`               | business logic, filesystem-heavy work             |
| Renderer      | `packages/desktop/src/renderer/`                                 | React, DOM, browser APIs                     | Node.js APIs, Electron main APIs                  |
| Common        | `packages/desktop/src/common/`                                   | pure/shared types, mappers, adapters         | code that assumes only one runtime unless guarded |
| Web host      | `packages/web-host/src/`                                         | Node/Bun HTTP, backend process lifecycle     | app business routes                               |

API bridge:

- `packages/desktop/src/common/adapter/httpBridge.ts` defines `httpGet`, `httpPost`, `httpPut`, `httpPatch`, `httpDelete`, `wsEmitter`, backend URL resolution, and structured `BackendHttpError`.
- `packages/desktop/src/common/adapter/ipcBridge.ts` is the main typed API surface consumed by renderer code. Most entries map to `deeporganiser-core` routes, not Electron IPC handlers.
- `packages/desktop/src/common/adapter/browser.ts` adapts `@office-ai/platform` bridge calls to Electron IPC in desktop mode or WebSocket in WebUI mode.
- `packages/desktop/src/common/adapter/main.ts` broadcasts bridge events to Electron BrowserWindows and WebSocket clients.
- `packages/desktop/src/common/config/configService.ts` reads and writes `/api/settings/client`; it is the renderer-facing settings cache.

When adding a new backend-backed UI feature, first add or reuse a typed wrapper in `ipcBridge.ts`, then call it from renderer hooks/components. Only add Electron bridge handlers in `packages/desktop/src/process/bridge/` for true desktop-only capabilities such as dialogs, windows, tray, OS paths, notifications, updates, and app lifecycle.

Recommended feature flow:

1. Identify the backend owner and route shape in `deeporganiser-core`/`ipcBridge.ts`.
2. Add or extend common DTOs/mappers under `packages/desktop/src/common/` when the renderer shape differs from the backend shape.
3. Keep data fetching in hooks or page-level modules; keep presentational components thin.
4. Subscribe to realtime changes through `wsEmitter`/`wsMappedEmitter` when the backend already emits events, instead of polling or inventing local state.
5. Add focused tests for mapper/pure logic first, then DOM/E2E coverage only when behavior spans the UI.

## Important Domains

Agents and ACP:

- Agent metadata comes from `/api/agents`; the frontend type is `AgentMetadata` in `packages/desktop/src/renderer/utils/model/agentTypes.ts`.
- Execution engine kinds are defined in `packages/desktop/src/common/types/agent/detectedAgent.ts`: `acp`, `remote`, `codex`, `openclaw-gateway`, and `nanobot`.
- ACP capability/config/mode/model/permission/session update types live in `packages/desktop/src/common/types/platform/acpTypes.ts`.
- ACP conversation API wrappers are under `ipcBridge.acpConversation`.
- Built-in and detected ACP CLIs include Claude Code, Codex, OpenCode, Qwen, Cursor Agent, Hermes Agent, Snow CLI, and others when available in PATH or contributed by extensions.
- Agent settings must use `/api/agents?include_disabled=true`; pickers should use `/api/agents` so disabled agents do not leak into normal selection.

Conversations:

- Conversation/message types live in `packages/desktop/src/common/config/storage.ts` and `packages/desktop/src/common/chat/chatLib.ts`.
- Renderer conversation UI is under `packages/desktop/src/renderer/pages/conversation/` with subareas for messages, preview, workspace, runtime, hooks, platform-specific views, and utilities.
- Database history requests are wrapped in `ipcBridge.database`, but persistence is handled by `deeporganiser-core`.

Providers and models:

- Provider management wrappers are `ipcBridge.mode.*`, backed by `/api/providers`.
- Shared API client/conversion helpers are in `packages/desktop/src/common/api/`.
- Do not store API keys in logs. `httpBridge` already redacts sensitive log keys; preserve that behavior.

MCP and skills:

- MCP server management is under `ipcBridge.mcpService`, backed by `/api/mcp/*`.
- Built-in MCP scripts are compiled by `scripts/build-mcp-servers.js` and included through `packages/desktop/electron.vite.config.ts` plus `electron-builder.yml`.
- Built-in image generation MCP code is in `packages/desktop/src/process/resources/builtinMcp/`.
- Skill marketplace/config UI lives in settings/capabilities areas; extension-contributed skills are surfaced through `/api/extensions/skills`.

Team mode:

- Team APIs are in `ipcBridge.team`, backed by `/api/teams/*`.
- Team UI lives in `packages/desktop/src/renderer/pages/team/`.
- Team persisted structures are typed in `packages/desktop/src/common/types/team/`.
- The database schema/migrations include `teams`, `mailbox`, and `team_tasks`; runtime orchestration is backend-owned.

Remote agents:

- Remote agent APIs are in `ipcBridge.remoteAgent`, backed by `/api/remote-agents/*`.
- Remote agent configs include protocol, auth, device identity, and `allow_insecure` fields.

Extensions:

- Extension APIs are under `ipcBridge.extensions` and `ipcBridge.hub`.
- Examples in `examples/hello-world-extension/` show contributed assistants, agents, ACP adapters, MCP servers, skills, themes, settings tabs, scripts, and assets.

Cron and channels:

- Scheduled task pages are under `packages/desktop/src/renderer/pages/cron/`.
- Cron persistence/runtime is backend-owned, surfaced through `/api/cron/*` wrappers in `ipcBridge.ts`.
- Channel assistant settings are represented in config keys such as `assistant.lark.*`, `assistant.dingtalk.*`, `assistant.weixin.*`, and `assistant.wecom.*`.

Desktop pet:

- Renderer pet pages live under `packages/desktop/src/renderer/pet/`.
- Preloads are `packages/desktop/src/preload/pet*.ts`.
- Main-process pet management is under `packages/desktop/src/process/pet/`.

## Data And Paths

Important path behavior:

- Electron data path is resolved by `getDataPath()` in `packages/desktop/src/process/utils/utils.ts`.
- On macOS, Electron creates CLI-safe symlinks such as `~/.deeporganiser-dev` and `~/.deeporganiser-config-dev` to avoid spaces in `Application Support` paths breaking external CLIs.
- Standalone WebUI intentionally defaults to `~/.openscience-web-dev` so it does not pre-create or block Electron's symlink targets.
- Legacy JSON/base64 file storage still exists in `packages/desktop/src/process/utils/initStorage.ts` for migration and some local config. New user-facing settings should normally go through backend `/api/settings/client`.
- SQLite schema helpers and legacy migration code under `packages/desktop/src/process/services/database/` are not the full backend database implementation. Treat them as compatibility/migration support unless the code path proves otherwise.

## Development Commands

Requirements:

- Node.js `>=22 <25` for this project. The app can fail in surprising ways on Node 25+ even if dependencies install.
- Bun is the package manager/runtime.
- Python 3.11+ is needed for native module compilation.
- A working `resources/bundled-deeporganiser-core/<platform>-<arch>` bundle is needed for Electron/WebUI startup unless you set `DEEPORGANISER_CORE_BIN` to a valid backend binary.

Common commands:

```bash
bun install
bun start                 # Electron dev app
bun run start:multi       # second isolated Electron dev instance
bun run webui             # standalone WebUI, builds first unless skipped
bun run webui -- --no-build --no-open
bun run package           # build main/preload/renderer to out/
bun run resetpass         # reset WebUI/admin password
```

Useful local run pattern for a detached second worktree:

```bash
screen -dmS deeporganiser-lark bash -lc 'cd /Users/yixuan/Documents/DeepScientist_lark && DEEPORGANISER_MULTI_INSTANCE=1 bun run start:multi > /tmp/deeporganiser-lark-run/electron-screen.log 2>&1'
screen -r deeporganiser-lark
screen -S deeporganiser-lark -X quit
tail -f /tmp/deeporganiser-lark-run/electron-screen.log
```

When two development checkouts are running, expect ports to shift. The first free Vite renderer port is commonly `5173`, CDP starts at `9230`, and `start:multi` uses isolated dev data such as `~/.deeporganiser-dev-2`.

Build/package commands:

```bash
bun run dist
bun run dist:mac
bun run dist:win
bun run dist:linux
node scripts/build-with-builder.js auto --pack-only --skip-native
```

Quality commands:

```bash
bun run lint
bun run lint:fix
bun run format
bun run format:check
bunx tsc --noEmit
bun run test
bun run test:coverage
bun run test:integration
bun run test:e2e
```

When changing i18n:

```bash
bun run i18n:types
node scripts/check-i18n.js
```

Before pushing, use `just push` rather than raw `git push`; it runs the project checks before pushing.

## Testing Guidance

Vitest 4 is configured in [vitest.config.ts](vitest.config.ts):

- Node tests: `tests/unit/**/*.test.ts`, `tests/integration/**/*.test.ts`, `tests/regression/**/*.test.ts`.
- DOM tests: `tests/unit/**/*.dom.test.ts(x)` under jsdom.
- Coverage includes `packages/desktop/src/**/*.{ts,tsx}` and `packages/**/src/**/*.{ts,tsx}` by default, with explicit exclusions for entrypoints, declarations, static assets, types, and JSON.

Playwright E2E is configured in [playwright.config.ts](playwright.config.ts):

- Test dir: `tests/e2e`
- Workers: `1` because Electron tests share an app instance.
- Use targeted E2E commands when possible, especially for team mode.

Write tests around observable behavior and failure paths. For new logic, separate pure logic from IO so it can be tested without launching Electron or `deeporganiser-core`.

Testing selection:

- Mapper, parser, config migration, and startup-classification changes: targeted Vitest unit tests.
- Renderer layout/state changes: DOM tests when selectors and state transitions matter; avoid E2E for pure styling.
- Electron startup, backend bridge, WebUI proxy, team/conversation flows: targeted Playwright E2E.
- DeepOrganiser Core bundle availability: run the resource verifier above and a `/health` smoke using the actual binary.
- i18n-visible renderer changes: regenerate i18n types and run `node scripts/check-i18n.js`.

## Code Conventions

File and directory structure:

- A single directory must not exceed 10 direct children. Split by responsibility before adding the 11th item.
- Do not create single-file directories.
- Use the `architecture` skill before creating files/modules or changing placement.
- Renderer component/feature directories use PascalCase when they represent a specific component/module. Categorical directories (`components`, `hooks`, `utils`, `services`, `pages`) are lowercase.
- Non-renderer directories are lowercase.
- Platform directories such as `acp`, `codex`, `gemini`, `nanobot`, and `openclaw` are lowercase everywhere.

Naming:

- React components/classes: PascalCase.
- Hooks: camelCase with `use` prefix.
- Utilities/helpers: camelCase.
- Types/constants/config files: camelCase.
- Constants inside files: UPPER_SNAKE_CASE.
- Unused parameters: prefix with `_`.
- Prefer `type` over `interface` unless extending/merging or matching existing interface-heavy code.

TypeScript:

- Strict mode is expected.
- Avoid `any`; use `unknown`, typed DTOs, or local narrowing.
- Use aliases such as `@/`, `@common/`, `@renderer/`, `@process/`, and `@worker/`.
- Keep comments in English for code comments. Existing Chinese comments may remain; avoid adding new mixed-language implementation comments unless needed for nearby consistency.

Formatting:

- Single quotes.
- Trailing commas in multi-line arrays/objects.
- Single-element arrays that fit on one line stay inline.
- Use `oxfmt`/`oxlint`; do not manually fight the formatter.

## UI And Styling Rules

- Use `@arco-design/web-react` components for interactive UI. Do not use raw `<button>`, `<input>`, `<select>`, `<textarea>`, or custom interactive primitives unless there is a specific low-level reason.
- Use `@icon-park/react` for icons. The Vite IconPark plugin wraps icon imports through `IconParkHOC`.
- Prefer UnoCSS utility classes for simple styles.
- Complex reusable component styles must use CSS Modules.
- Do not hardcode colors in normal component styles. Use semantic Uno tokens from [uno.config.ts](uno.config.ts) or CSS variables. Theme preset files are the exception because they define tokens.
- Global CSS belongs only under `packages/desktop/src/renderer/styles/`.
- Component-scoped Arco overrides should live in CSS Modules using `:global(...)`, not in new global CSS files.
- All user-facing text must use i18n. Read `packages/desktop/src/common/config/i18n-config.json` before changing locale files and add keys to every supported language.

## Local Skills

Use these repository skills when their triggers apply:

- `architecture`: creating files/modules, placing code, changing architecture, adding services/bridges/agents/workers.
- `i18n`: adding or changing user-facing text or locale modules.
- `testing`: writing tests, changing tested logic, or before claiming feature work is complete.
- `bump-version`: release version bump flow only.

Skill files are under `.claude/skills/`. If a skill references additional files, read them before acting.

## Common Pitfalls

- Do not add business logic to `packages/web-host/src/static-server.ts`; it is a proxy/static server.
- Do not bypass `ipcBridge.ts` from renderer code for backend business calls unless the local pattern already does so for a narrow reason.
- Do not import `@process/*` into renderer code.
- Do not assume `docs/architecture/overview.md` exists in this clone; inspect source and current docs together.
- Do not share Electron and standalone WebUI default data dirs unless the user explicitly asks for it.
- Do not overwrite `resources/bundled-deeporganiser-core` casually; packaging and local WebUI depend on the platform-specific binary layout.
- Do not commit `resources/bundled-deeporganiser-core`; it is a local/prepared artifact and is ignored on purpose.
- Do not fix an DeepOrganiser Core missing-resource error by hiding the installation integrity dialog. Fix the bundle, resolver, or packaging path.
- Do not log API keys, auth tokens, JWTs, or provider secrets.
- Do not treat disabled agents as available in picker UI. Use the managed agents endpoint only for settings.
- Do not run broad destructive cleanup commands in user data dirs. The app stores real conversations, providers, credentials, cron jobs, and team state there.
- Do not add AI signatures such as `Co-Authored-By` or generated-by footers to commits or PRs.

## Useful Inspection Commands

```bash
rg --files | sed -n '1,200p'
rg -n "ipcBridge\\.|/api/agents|/api/teams|/api/providers|wsEmitter" packages/desktop/src
rg -n "CREATE TABLE|ALTER TABLE|CURRENT_DB_VERSION" packages/desktop/src/process/services/database
rg -n "model|provider|agent_type|backend" packages/desktop/src/common packages/desktop/src/renderer
curl -fsS http://127.0.0.1:25809/api/auth/status
curl -fsS 'http://127.0.0.1:25809/api/agents?include_disabled=true'
```

When the WebUI was launched through this workspace during setup, it used:

```bash
DEEPORGANISER_DATA_DIR="$HOME/.openscience-web-dev" DEEPORGANISER_OPEN_BROWSER=0 DEEPORGANISER_NO_BUILD=1 bun run webui -- --no-open --no-build
```

The launchd label used for that local service was `com.codex.deeporganiser.webui`; remove it with `launchctl remove com.codex.deeporganiser.webui` if you need to stop that background instance.

---
> Source: [ResearAI/OpenScience](https://github.com/ResearAI/OpenScience) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
