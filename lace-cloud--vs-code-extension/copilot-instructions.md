## vs-code-extension

> This document is for AI coding agents working on the Lace codebase. It contains the architecture, conventions, and constraints you must follow.

# AGENTS.md -- Lace VS Code Extension

This document is for AI coding agents working on the Lace codebase. It contains the architecture, conventions, and constraints you must follow.

## What This Project Is

Lace VS Code Extension -- a visual Terraform composer. Users browse modules from a registry sidebar, drag them onto a ReactFlow canvas, wire inputs/outputs, configure settings, and generate `.tf` files. The extension talks to a Go CLI binary (`lace`) over gRPC on an ephemeral TCP port.

The extension is an **opaque rendering layer**. It does NOT understand Terraform IR -- no Bundle, no Module, no Binding types. All IR knowledge lives in the CLI. The extension receives a `CanvasView` (a view model of nodes, edges, errors) from the CLI and renders it. All mutations go through the CLI via RPC.

## Build & Test

```bash
npm run compile          # Rspack build (host + webview)
npm run test:unit        # 26 unit tests (vitest, no binary needed)
npm run test:e2e         # 6 E2E tests (requires lace + terraform binaries)
npm run test:watch       # Watch mode (unit tests)
npx tsc --noEmit         # Type-check only -- must be zero errors
```

Always run `npm run test:unit` after any change. All 26 unit tests must pass. TypeScript must compile with zero errors.

E2E tests spawn a real `lace module serve` process and exercise the full gRPC transport. They require the `lace` binary and `terraform` to be installed. Set `LACE_BINARY=/path/to/lace` to override the default PATH lookup.

## Architecture Overview

### Host side (Node.js, VS Code API)

- **`extension.ts`** -- activation, command registration, server lifecycle, `lace.login` command (GitHub PAT input → auth/login RPC)
- **`createWebviewPanel.ts`** -- manages canvas panel lifecycle, routes PostMessage between webview and CLI RPC, handles auto-save (debounced ~2s after last edit), dirty tracking, module drops from sidebar, three-phase generate orchestration (generate → fmt → validate with progress messages)
- **`server-manager.ts`** -- manages the CLI engine server process (`lace module serve`), spawns/restarts the child process, reads `LACE_GRPC_PORT=<port>` from stdout, exposes `rpcClient` (gRPC `LaceClient`), checks auth status after init (emits `'auth'` event), exposes `login(token)` and `checkAuth()` methods
- **`grpc-client.ts`** -- gRPC client wrapping the generated `LaceEngineClient`, promisifies callbacks, converts proto types to extension render types
- **`RegistrySidebarProvider.ts`** -- WebviewViewProvider for the registry sidebar, fetches module data, posts `addToCanvas` messages to canvas
- **`ModuleDetailPanel.ts`** -- detail tab for inspecting registry modules

### Webview side (React, browser environment)

- **`App.tsx`** -- root component, creates `PostMessageEngine`, listens for `HostToWebview` messages, routes `engineResult` to engine's pending promises, forwards `loadState` to context
- **`Canvas.tsx`** -- renders the `CanvasView` using `CompositeEditor` (ReactFlow), handles user interactions (connect, delete, drag, save, generate, undo/redo), delegates all mutations to engine
- **`engine-context.ts`** -- React context providing `CanvasState` (view, loading, error, generation) + `CanvasEngine` + `updateView`
- **`post-message-engine.ts`** -- implements `CanvasEngine` by sending postMessages to the host, which forwards them to the CLI via gRPC

### Chat participant (`@lace`)

- **`chat/participant.ts`** -- registration + agentic tool loop (up to 15 rounds)
- **`chat/tools/`** -- five tool groups: registry, graph-read, graph-write, workspace, generate
- **`chat/system-prompt.ts`** -- system prompt injected into every chat request

## Protocol

Defined in `src/types/protocol.ts`. Messages are discriminated unions on `command`.

### Host to Webview (`HostToWebview`)

| Command            | Payload                                         | Purpose                             |
| ------------------ | ----------------------------------------------- | ----------------------------------- |
| `loadState`        | `state: CanvasView`                             | Send full view model to render      |
| `generateProgress` | `phase: GeneratePhase`                          | Step-by-step generate progress      |
| `generateSuccess`  | `files?: string[]`                              | Generation succeeded                |
| `generateError`    | `message: string`, `diagnostics?: Diagnostic[]` | Generation failed                   |
| `engineResult`     | `requestId: string`, `result?`, `error?`        | Response to an `engineCall` request |

### Webview to Host (`WebviewToHost`)

| Command        | Payload                                  | Purpose                                    |
| -------------- | ---------------------------------------- | ------------------------------------------ |
| `webviewReady` | (none)                                   | Webview initialized, ready for `loadState` |
| `engineCall`   | `requestId: string`, `method`, `params?` | RPC call forwarded to CLI engine           |
| `markDirty`    | (none)                                   | Canvas has unsaved changes                 |
| `markClean`    | (none)                                   | Canvas is clean (matches saved state)      |

## Engine Interface

`CanvasEngine` in `src/webview/engine.ts` defines what the webview can ask the host to do. `PostMessageEngine` implements it by sending `engineCall` postMessages.

**Session methods:** `sessionOpen`, `sessionSave`, `sessionClose`, `sessionGenerate`

**Action methods (return updated `CanvasView`):** `dropModule`, `connect`, `autoConnect`, `disconnect`, `updateInput`, `updateAllInputs`, `renameInstance`, `deleteInstance`, `setVariables`, `setExports`, `setLocals`, `undo`, `redo`

**Action methods (void):** `syncLayout`, `setTerraform`, `setProviders`, `setDependsOn`, `setEnvironments`

**Query methods:** `queryNodeConfig`, `queryEdgeConfig`, `querySettings`, `queryGraphSummary`, `queryValidate`

**Auth methods (always available, no API client required):** `authStatus`, `authLogin`, `authLogout`

## Type System

### `CanvasView` (render.ts) -- the view model

```
CanvasView { module_name, nodes: RenderNode[], edges: RenderEdge[], errors: RenderError[], can_undo, can_redo, is_dirty }
RenderNode { id, label, kind, module_key?, position, has_errors, error_messages }
RenderEdge { id, source, target, source_output, target_input }
```

### Query response types (render.ts)

- `NodeConfig` -- inputs, outputs, siblings, depends_on for a node config panel
- `EdgeConfig` -- source outputs + target unbound inputs for edge config panel
- `SettingsConfig` -- terraform, providers, locals, environments

### Protocol types (protocol.ts)

- `HostToWebview`, `WebviewToHost` -- message unions
- `GeneratePhase` -- `'generating' | 'formatting' | 'validating'`
- `Diagnostic` -- RPC validation diagnostic (severity, message, file, line, column)
- `RegistryModule` -- shared across sidebar and detail panel

### Context state (engine-context.ts)

```
CanvasState { view: CanvasView | null, loading: boolean, error: string | null, generation: number }
```

## File Organization

| Layer           | Path                                          | Purpose                                            |
| --------------- | --------------------------------------------- | -------------------------------------------------- |
| Entry point     | `extension.ts`                                | Activation, command registration                   |
| Host: canvas    | `webview/createWebviewPanel.ts`               | Panel lifecycle, PostMessage routing, auto-save    |
| Host: HTML      | `webview/getWebviewContent.ts`                | Webview HTML shell, keyboard handlers              |
| Host: detail    | `webview/ModuleDetailPanel.ts`                | Registry module detail tab                         |
| Host: sidebar   | `containers-views/RegistrySidebarProvider.ts` | Registry sidebar provider                          |
| Host: engine    | `utilities/engine/server-manager.ts`          | CLI process lifecycle                              |
| Host: RPC       | `utilities/engine/grpc-client.ts`             | gRPC client (LaceClient)                           |
| Host: errors    | `utilities/engine/rpc-errors.ts`              | RPC error handling helpers                         |
| Host: terraform | `utilities/terraform.ts`                      | Terraform CLI helpers (fmt, validate, isAvailable) |
| Webview: root   | `webview/App.tsx`                             | Message listener, engine creation, context         |
| Webview: canvas | `webview/Canvas.tsx`                          | ReactFlow rendering, user interaction handlers     |
| Webview: engine | `webview/engine.ts`                           | CanvasEngine interface                             |
| Webview: impl   | `webview/post-message-engine.ts`              | PostMessageEngine (CanvasEngine over postMessage)  |
| Webview: state  | `webview/state/engine-context.ts`             | React context (CanvasState + engine)               |
| Webview: types  | `webview/types/render.ts`                     | CanvasView, RenderNode, RenderEdge, configs        |
| Shared types    | `types/protocol.ts`                           | HostToWebview, WebviewToHost, Diagnostic           |
| Components      | `webview/components/`                         | UI: ModuleNode, panels, ActionBar, SlidePanel      |
| Utilities       | `webview/utils/`                              | identifiers, parseValue                            |
| Styles          | `webview/styles/panel.ts`                     | Shared panel style constants                       |
| Chat            | `chat/`                                       | @lace chat participant + tools                     |
| Tests           | `webview/__tests__/`                          | Unit + E2E tests                                   |

## Testing

| File                          | What it tests                                                       |
| ----------------------------- | ------------------------------------------------------------------- |
| `chat-tools.test.ts`          | Chat participant graph-write, graph-read, generate tools (13 tests) |
| `registry-tools.test.ts`      | Chat participant registry search and inspect (5 tests)              |
| `parseValue.test.ts`          | HCL literal parsing (5 tests)                                       |
| `identifiers.test.ts`         | Terraform identifier validation (3 tests)                           |
| `e2e-rpc-integration.test.ts` | Full CLI RPC pipeline, generate to disk (6 tests, E2E)              |

## Critical Invariants

1. **Extension is a rendering layer.** No IR knowledge, no Terraform parsing, no Bundle/Module types. The extension receives `CanvasView` and renders it.

2. **All mutations go through engine interface.** Webview calls `CanvasEngine` methods, `PostMessageEngine` sends `engineCall` postMessages to the host, host forwards to CLI via gRPC. Never modify state directly.

3. **`CanvasView` is the single source of truth** for what gets rendered. The CLI projects IR into this view model.

4. **`generation` counter controls fitView.** `CompositeEditor` remounts when `generation` changes (via React `key`), triggering `fitView` on fresh state loads.

5. **Dirty tracking flows from CLI.** `is_dirty` is a field on `CanvasView` set by the CLI engine. The webview posts `markDirty`/`markClean` to the host for VS Code's document dirty indicator.

6. **Auto-save in host fires ~2s after last edit** (debounced). Don't add additional debouncing in the webview.

7. **`canvasSave` custom event** bridges VS Code Cmd+S to the webview's `engine.sessionSave()`. Dispatched from `getWebviewContent.ts` keyboard handler.

8. **All RPC to CLI goes through the host's server-manager**, never directly from the webview. The webview only talks to the host via postMessage.

9. **Two Rspack entries.** `out/extension.js` (Node.js, CommonJS) and `out/webview.js` (browser). Don't import VS Code APIs in webview code or DOM APIs in host code.

10. **Registry browsing is host-side.** The webview never fetches registry data. Module drops arrive as `loadState` messages from the host after the host calls `action/drop_bundle` via gRPC.

11. **Auth is runtime-swappable.** The server checks `auth/status` after initialization and emits an `'auth'` event. If credentials are missing, registry is unavailable but the extension still works. Users can run `lace.login` to authenticate from VS Code — the server hot-swaps its API client without restarting.

## Common Gotchas

- **PostMessage is async and untyped at runtime.** `protocol.ts` types are the contract -- runtime messages are plain objects. Always handle unknown/missing fields defensively in message handlers.

- **`generation` must increment on every `loadState`** or fitView won't trigger, because `CompositeEditor` uses it as a React `key`.

- **`is_dirty` comes from the CLI engine**, not tracked locally. The webview reads it from `CanvasView` and posts `markDirty`/`markClean` to the host.

- **`canvasSave` event is dispatched from `getWebviewContent.ts`** keyboard handler, not from within React. Canvas listens for it via `window.addEventListener`.

- **`__canvasUndo` window global** is needed by the HTML-level Cmd+Z keyboard listener in `getWebviewContent.ts`. It's set by Canvas and points to the `onUndo` callback.

- **ReactFlow v12.** This project uses `@xyflow/react` v12. Node components receive `NodeProps<Node<TData, TType>>`.

- **Tailwind CSS v4** for all styling. No separate CSS files for components.

- **`engineResult` routing.** `App.tsx` receives `engineResult` messages and calls `engine.handleResult()` to resolve the pending promise in `PostMessageEngine`. This is the async RPC bridge.

## Style Guide

- TypeScript strict mode. No `any` except at RPC boundaries.
- Function components with hooks. No class components.
- Descriptive test names that state the behavior, not the implementation.
- No dead code. Every export is consumed. Run `npx tsc --noEmit` -- zero errors.

## Dependencies

**Runtime:** `react`, `react-dom`, `@xyflow/react` -- that's it.

**Dev:** `typescript`, `vitest`, `rspack`, `tailwindcss`, `postcss`, `prettier`, `husky`, `lint-staged`.

Do not add native dependencies (sqlite3, node-gyp, etc.). The extension has zero native Node.js modules. The CLI handles all heavy lifting.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lace-cloud)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lace-cloud)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
