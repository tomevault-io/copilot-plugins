## 005-hermes-desktop-domain

> Hermes Desktop domain rules for Cursor. Defines product boundaries, domain vocabulary, module ownership, profile runtime rules, Web Operator rules, Windows deployment rules, and implementation guardrails.


# 005 - Hermes Desktop Domain Rules

## 1. Product Identity

This project is Hermes Desktop / Portal Desktop.

It is an Electron desktop shell for operating, configuring, observing, and extending a local or remote Python-based `hermes-agent`.

It is not the LLM runtime.
It is not the tool execution engine.
It is not the memory engine.
It is not the Hermes Gateway protocol implementation.

The desktop app owns:

- Electron window lifecycle
- Renderer UI
- Preload bridge
- IPC routing
- Local configuration UX
- Local filesystem orchestration
- Local process management
- Gateway lifecycle control
- Profile runtime control plane
- Web Operator desktop bridge
- Installer / bootstrap / diagnostics UI
- Windows desktop deployment experience

The Python `hermes-agent` owns:

- LLM inference routing
- Tool execution
- Memory retrieval
- Skill execution
- Gateway API behavior
- `/v1/chat/completions`
- agent-side reasoning and orchestration

Do not move Python backend responsibilities into Electron.

---

## 2. Core Process Boundary

The system has four runtime layers:

```text
Renderer Process
  React UI only
  No Node.js access
  Calls window.hermesAPI only

Preload Bridge
  Exposes typed hermesAPI
  Security boundary
  No business UI

Main Process
  Node.js privileged layer
  Owns IPC handlers
  Owns filesystem/process/SQLite/gateway lifecycle

Python Gateway
  External process
  Treated as black box
  Accessed through local HTTP/SSE or CLI fallback
```

Hard rules:

- Renderer must never import `electron`, `fs`, `path`, `child_process`, `better-sqlite3`, `os`, or Node-only modules.
- Renderer must never call `ipcRenderer` directly.
- Renderer must only use `window.hermesAPI`.
- Preload is the only bridge.
- Main Process is the only place for filesystem, SQLite, local process, Git, Python, NSIS, PATH, and installer operations.
- Python Gateway must be treated as an external service, not as in-process code.

---

## 3. Domain Vocabulary

Use these domain names consistently.

### App / Shell

- `Hermes Desktop`
- `Portal Desktop`
- `Desktop Shell`
- `Desktop Runtime`
- `Desktop Control Plane`

### Backend

- `hermes-agent`
- `Python Gateway`
- `Hermes Gateway`
- `Gateway Runtime`

### Profile

- `default profile`
- `specialist profile`
- `profile runtime`
- `profile home`
- `profile workspace`
- `profile gateway`
- `profile runtime db`

### Workspace

- `Portal Home`
- `Profile Workspace`
- `Runtime Center`
- `Web Operator`
- `Observability`
- `Local Install`
- `Settings`

### Automation

- `Desktop Tool Bridge`
- `BrowserController`
- `WebContentsView`
- `Browser IPC`
- `Web Operator Action`
- `Sensitive Action Confirmation`
- `DOM Snapshot`
- `Screenshot History`

### Windows Deployment

- `NSIS assisted installer`
- `Install Directory`
- `Runtime Root`
- `User PATH`
- `System PATH`
- `Bootstrap`
- `Local Doctor`
- `Install Log`

Do not invent alternative domain names unless explicitly requested.

---

## 4. Module Ownership

Respect the existing module responsibilities.

### `src/main/index.ts`

Single IPC registration hub.

Add new IPC handlers here only after implementing the domain logic in a separate `src/main/*.ts` module.

Pattern:

```ts
ipcMain.handle("domain:action", async (_, input) => {
  return domainAction(input);
});
```

Do not place large business logic directly inside `setupIPC()`.

---

### `src/preload/index.ts`

Authoritative renderer API surface.

Every Renderer capability must be declared here first.

Pattern:

```ts
const hermesAPI = {
  getRuntimeStatus: () =>
    ipcRenderer.invoke("runtime:get-status"),
};
```

Every long-lived event listener must return an unsubscribe function.

Pattern:

```ts
onInstallProgress: (callback) => {
  const listener = (_event, payload) => callback(payload);
  ipcRenderer.on("install-progress", listener);
  return () => ipcRenderer.removeListener("install-progress", listener);
};
```

---

### `src/preload/index.d.ts`

Runtime contract for the Renderer.

Every new `window.hermesAPI` method must have a matching TypeScript declaration.

Do not use `any`.
Use shared types from `src/shared/**`.

---

### `src/main/hermes.ts`

Gateway lifecycle module.

Allowed responsibilities:

- `startGateway`
- `stopGateway`
- `restartGateway`
- gateway health polling
- send message to Gateway
- SSE stream handling
- CLI fallback
- Gateway config injection before start

Do not put UI state here.
Do not put Renderer concerns here.
Do not modify profile runtime DB here unless explicitly part of gateway lifecycle.

---

### `src/main/installer.ts`

One-time installation and local environment bootstrap.

Allowed responsibilities:

- detect install status
- create Python venv
- install dependencies
- run doctor
- run update
- read installer logs
- manage bootstrap progress
- import/export backup

Do not put profile runtime governance here unless it belongs to initial bootstrap.

---

### `src/main/config.ts`

Profile-aware `.env` and `config.yaml` management.

Allowed responsibilities:

- model config
- provider config
- API key references
- local / remote mode
- platform toggles
- toolset toggles
- config cache

Any config mutation that affects Gateway behavior must trigger gateway restart.

---

### `src/main/utils.ts`

`profileHome(profile?)` is the single route for profile-scoped filesystem paths.

Every module reading or writing profile data must use `profileHome()` or a higher-level abstraction that uses it.

Do not hardcode:

```text
~/.hermes
%USERPROFILE%\.hermes
C:\Users\...\ .hermes
```

inside feature modules.

---

### `src/main/sessions.ts`

Owns SQLite `state.db` session reads.

Do not access `state.db` from Renderer.
Do not open SQLite from UI components.
Do not mix profile session data across profiles.

---

### `src/main/profiles.ts`

Owns profile CRUD and profile discovery.

Profile-scoped features must accept `profile?: string` and pass it through the full IPC stack.

---

### `src/main/skills.ts`

Owns skill discovery and skill metadata.

Electron is not a plugin host.
Skills are Hermes / Python-side capabilities, not Electron plugins.

Do not implement skills as Electron extensions unless explicitly requested.

---

## 5. Extension Flow

### New Main Capability

When adding a new backend capability:

1. Define types in `src/shared/<domain>/`.
2. Implement domain logic in `src/main/<domain>.ts`.
3. Register IPC in `src/main/index.ts`.
4. Expose wrapper in `src/preload/index.ts`.
5. Add declaration in `src/preload/index.d.ts`.
6. Use only `window.hermesAPI` from Renderer.
7. Add loading/error handling in UI.
8. Add typecheck-safe tests or manual acceptance notes.

Never skip Preload.
Never call `ipcRenderer` from Renderer.
Never use `any` in IPC input/output types.

---

### New UI Screen

When adding a new screen:

1. Add or reuse a `View` value in `Layout.tsx`.
2. Add navigation only if the screen is user-facing.
3. Place screen under `src/renderer/src/screens/<ScreenName>/`.
4. Place reusable components under `src/renderer/src/components/<domain>/`.
5. Place data hooks under `src/renderer/src/hooks/`.
6. Use shared types from `src/shared/`.
7. Implement loading, empty, error, and ready states.
8. Keep screen orchestration separate from presentational components.

Do not create a second root app.
Do not bypass the existing Layout shell.
Do not create a new BrowserWindow unless explicitly requested.

---

## 6. Profile Runtime Domain Rules

Profiles are isolated runtime workspaces.

Default profile:

```text
~/.hermes/
```

Named profile:

```text
~/.hermes/profiles/<profileId>/
```

Every profile-scoped feature must:

- accept `profile?: string`
- pass `profile` from Renderer to Preload to Main
- resolve paths through `profileHome(profile)`
- avoid mixing sessions, skills, memory, config, and runtime logs across profiles
- write audit events when changing profile runtime state

Never hardcode profile-specific paths.

---

## 7. Multi Profile Runtime Rules

The product direction is local multi-profile runtime.

Expected capabilities:

- multiple gateway instances
- profile-specific ports
- profile-specific status
- profile-specific logs
- gateway health polling
- auto restart
- port conflict detection
- runtime DB migration
- runtime status recovery
- profile workspace entry

When implementing multi-profile runtime:

- keep `default` as Portal control entry
- keep specialist profiles as separate workspaces
- do not merge profile `state.db`
- do not share profile memory by default
- do not share credentials by default
- use explicit context share or delegation events for cross-profile collaboration

---

## 8. Delegation Domain Rules

Delegation means default profile or another controller profile dispatches work to specialist profiles.

Delegation must be task-like and auditable.

Use explicit state names:

```ts
type DelegationTaskStatus =
  | "created"
  | "dispatching"
  | "running"
  | "succeeded"
  | "failed"
  | "timeout"
  | "cancelled";
```

Delegation records should include:

- `taskId`
- `sourceProfile`
- `targetProfile`
- `status`
- `input`
- `result`
- `error`
- `createdAt`
- `updatedAt`
- `contextRefs`
- `auditRefs`

Do not implement delegation as invisible direct function calls.
Do not let profiles silently share state.

---

## 9. Web Operator Domain Rules

Web Operator is a desktop automation subsystem.

It must be profile-aware.

Every browser action should carry:

```ts
interface BrowserToolRequest {
  profileId: string;
  source: "user" | "hermes" | "system";
  toolName: string;
  args: Record<string, unknown>;
  requireConfirm?: boolean;
}
```

Profile-specific browser session partition pattern:

```text
default  -> persist:aios-web-default
coding   -> persist:aios-web-coding
finance  -> persist:aios-web-finance
writer   -> persist:aios-web-writer
```

Rules:

- Web login state must not leak across profiles.
- Browser actions must be auditable.
- Sensitive actions require confirmation.
- Screenshots must be associated with profile and task id when available.
- DOM snapshots should not be stored without size limits.
- Never execute browser automation from Renderer directly.
- Web Operator control belongs to Main Process / BrowserController / Desktop Tool Bridge.

---

## 10. Observability Domain Rules

Every runtime feature must expose enough state for diagnosis.

For Gateway Runtime:

- status
- port
- pid
- profileId
- health
- last error
- restart count
- stdout/stderr log path
- last started at
- last stopped at

For installer/bootstrap:

- phase
- progress
- current command
- log path
- error code
- retry action

For Web Operator:

- action id
- profile id
- URL / domain
- action type
- status
- confirmation status
- screenshot ref
- DOM snapshot ref
- error

Do not implement silent background operations.
Every long-running operation must expose progress or logs.

---

## 11. Local Install / Windows Deployment Rules

The Windows deployment target is internal Windows 10+ desktop usage.

Preferred install behavior:

- NSIS assisted installer
- user-selectable install directory
- per-user install by default
- write User PATH by default
- System PATH only when admin mode is explicit
- installation logs are written locally
- Hermes Agent source can be local zip or Git clone
- actual Python setup should run in Electron Bootstrap, not inside NSIS

Recommended runtime layout:

```text
$INSTDIR/
  Hermes Desktop.exe
  resources/
  bin/
    hermes-desktop.cmd
    hermes.cmd
  runtime/
    hermes-agent/
    logs/
    cache/
    downloads/
```

Recommended user config layout:

```text
%USERPROFILE%\.hermes/
  config.yaml
  .env
  state.db
  SOUL.md
  memories/
  skills/
  desktop/
    profile-runtime.db
    profile-runtime.yaml
  profiles/
```

Rules:

- Do not store secrets in `electron-builder.yml`.
- Do not package `.env`.
- Do not put private Git token in Renderer.
- Do not assume admin privileges.
- Do not assume Git/Python/uv already exist.
- Do not write to `Program Files` unless per-machine install is explicit.
- Do not run long Python dependency installation inside NSIS.
- Use Electron Bootstrap UI for Git clone, zip extraction, venv creation, dependency install, and diagnostics.

---

## 12. Storage Rules

Default Hermes layout:

```text
~/.hermes/
  config.yaml
  .env
  state.db
  SOUL.md
  memories/MEMORY.md
  skills/<category>/<name>/
  jobs.json
  desktop/sessions.json
  profiles/<name>/
```

Storage rules:

- `config.yaml` stores model/provider/tool/platform config.
- `.env` stores secret references and local tokens.
- `state.db` stores sessions and message search index.
- `SOUL.md` stores persona/system prompt.
- `MEMORY.md` stores flat-file memory.
- `skills/` stores installed skills.
- `desktop/` stores desktop-specific state/cache/runtime metadata.
- `profiles/<name>/` mirrors the default profile structure.

Do not mix desktop-only state into Hermes runtime config unless necessary.
Prefer `~/.hermes/desktop/` for desktop control-plane state.

---

## 13. Memory Domain Rules

`MEMORY.md` is a flat-file store.

Rules:

- Entries are delimited by `§`.
- Total memory text has a hard cap.
- Do not change delimiter without migration.
- Do not store large documents in MEMORY.md.
- Do not use memory for logs or task history.
- Do not mix memory across profiles unless explicit context sharing is implemented.

---

## 14. Skill Domain Rules

Skills are modular capability packages discovered from `SKILL.md`.

Rules:

- Skill metadata must be parsed from frontmatter.
- Skill path should preserve category/name structure.
- Bundled skills live in app resources.
- Installed skills live under profile home.
- Skill copy/sync should create backup before overwrite.
- Skill governance should track version/checksum when available.

Do not implement Electron-side skill execution.
Do not treat skills as npm plugins.

---

## 15. UI Domain Rules

The UI must reflect the product domain.

Primary screens:

```text
AIOSHome
ProfileWorkspace
RuntimeCenter
WebOperator
Observability
LocalInstall
Settings
```

Preferred page patterns:

### AIOSHome

- profile status cards
- recent tasks
- recent delegations
- pending confirmations
- Web Operator shortcut
- runtime health summary

### ProfileWorkspace

- current profile status
- last session
- skill list
- context list
- runtime logs
- gateway controls

### RuntimeCenter

- gateway instances table
- port/pid/status
- health polling
- restart/stop/start controls
- runtime DB status

### WebOperator

- current browser workspace
- action timeline
- confirmation queue
- screenshot/DOM panel
- profile partition indicator

### Observability

- gateway logs
- runtime events
- delegation timeline
- Web Operator audit
- skill sync history
- context share history
- diagnostics

### LocalInstall

- install directory
- runtime root
- agent source: local zip / git clone
- Git/Python/uv status
- PATH status
- bootstrap progress
- install logs

UI rules:

- Always show profile context when relevant.
- Always show runtime status when controlling gateway behavior.
- Long-running tasks must show progress.
- Errors must show actionable remediation.
- Dangerous actions must require confirmation.
- Logs should use monospace.
- Technical configuration should be grouped into cards or panels.
- Do not create generic SaaS dashboard UI unrelated to the desktop runtime.

---

## 16. Status Naming Rules

Use explicit status enums.

Gateway status:

```ts
type GatewayStatus =
  | "idle"
  | "starting"
  | "running"
  | "stopping"
  | "stopped"
  | "failed"
  | "unknown";
```

Install status:

```ts
type InstallStatus =
  | "not_installed"
  | "checking"
  | "installing"
  | "installed"
  | "failed"
  | "updating";
```

Doctor item status:

```ts
type DoctorStatus =
  | "pass"
  | "warning"
  | "error"
  | "skipped";
```

Web Operator action status:

```ts
type WebOperatorActionStatus =
  | "planned"
  | "pending_confirmation"
  | "running"
  | "succeeded"
  | "failed"
  | "cancelled";
```

Do not use ambiguous booleans like `isOk`, `done`, `active` for domain state when a typed status is more accurate.

---

## 17. Security and Policy Rules

Security-sensitive areas:

- API keys
- Git tokens
- PATH mutation
- shell execution
- Python dependency install
- browser automation
- credential forms
- system profile config
- finance/ERP/bank domains
- destructive file operations

Rules:

- Do not expose secrets to Renderer unless masked.
- Do not log tokens.
- Do not store Git credentials in plain UI state.
- Do not execute arbitrary shell strings from user input.
- Prefer `execFile` / `spawn` with args over shell string concatenation.
- Browser actions that type, submit, pay, delete, upload, or change credentials require confirmation.
- Profile policies should block unauthorized tools and domains.
- Every blocked action should expose a clear reason.

---

## 18. Coding Style Rules

TypeScript:

- No `any` in IPC contracts.
- Use shared interfaces from `src/shared/**`.
- Prefer discriminated unions for runtime states.
- Keep domain types close to domain folders.
- Do not duplicate the same type across Main/Renderer.

React:

- Keep screen orchestration in screen component.
- Move reusable UI blocks to components.
- Move data loading to hooks.
- Clean up event listeners on unmount.
- Do not put long business workflows inside JSX.
- Do not make components larger than necessary.

Main Process:

- Use domain modules.
- Keep IPC handlers thin.
- Validate inputs.
- Return typed results.
- Include error codes for operational failures.

---

## 19. Cursor Task Behavior

When asked to implement a feature, first classify it:

```text
UI-only
Renderer + existing hermesAPI
New IPC capability
Main Process domain feature
Installer/bootstrap feature
Profile Runtime feature
Web Operator feature
Windows deployment feature
```

Before coding, produce:

1. File change list
2. Domain boundary analysis
3. Required types
4. IPC additions if needed
5. UI states if UI is involved
6. Acceptance checklist

Do not modify unrelated modules.
Do not refactor global architecture unless explicitly requested.
Do not introduce new runtime dependencies without stating why.
Do not implement multiple large features in one pass.

---

## 20. Acceptance Checklist

Any completed change must satisfy:

- TypeScript typecheck passes.
- Renderer does not import Node.js modules.
- Renderer only calls `window.hermesAPI`.
- New IPC methods are typed in Preload and `.d.ts`.
- Profile-scoped operations pass `profile?: string`.
- Filesystem paths route through `profileHome()` or approved runtime path resolver.
- Gateway-affecting config changes restart Gateway.
- Long-running operations show progress or logs.
- Event listeners provide unsubscribe and are cleaned up.
- Errors include actionable messages.
- No secrets are logged.
- No fake data remains in production paths.
- UI includes loading, empty, error, and success states.

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
