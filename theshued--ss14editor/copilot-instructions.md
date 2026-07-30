## ss14editor

> description: "Rules for editing the API surface in src/Api/ — ApiRouter dispatcher and the per-domain partial-class handler files (StatusApi, FileApi, FolderApi, AssetApi, etc.). Covers route registration, README sync, path safety, and tests."

﻿---
description: "Rules for editing the API surface in src/Api/ — ApiRouter dispatcher and the per-domain partial-class handler files (StatusApi, FileApi, FolderApi, AssetApi, etc.). Covers route registration, README sync, path safety, and tests."
applyTo: "src/Api/**/*.cs"
---

# ApiRouter — Endpoint Rules

`ApiRouter` is a `partial class` split across `src/Api/`:

| File | Endpoints |
|------|-----------|
| [ApiRouter.cs](../../src/Api/ApiRouter.cs) | dispatcher + route registration table (no handlers) |
| [StatusApi.cs](../../src/Api/StatusApi.cs) | `/api/status`, `/api/configure` |
| [TreeApi.cs](../../src/Api/TreeApi.cs) | `/api/tree`, `/api/metadata`, `/api/proto-index`, `/api/search-protos`, `/api/refresh-index` |
| [FileApi.cs](../../src/Api/FileApi.cs) | `/api/file`, `/api/rename-file`, `/api/delete-file`, `/api/create-file` |
| [FolderApi.cs](../../src/Api/FolderApi.cs) | `/api/create-folder`, `/api/rename-folder`, `/api/delete-folder` |
| [AssetApi.cs](../../src/Api/AssetApi.cs) | `/api/texture`, `/api/texture-browse`, `/api/audio`, `/api/audio-browse` |
| [SourceApi.cs](../../src/Api/SourceApi.cs) | `/api/open-in-explorer`, `/api/open-default`, `/api/open-source` |
| [EventsApi.cs](../../src/Api/EventsApi.cs) | `/api/events` (SSE) |

## Adding a new `/api/*` endpoint

1. **Register the route** in the `_routes` dictionary inside the `ApiRouter` constructor in [ApiRouter.cs](../../src/Api/ApiRouter.cs). Keys must start with `/api/` and use kebab-case (`/api/refresh-index`, not `/api/refreshIndex`). Add it under the comment header for the matching domain.
2. **Add the handler method** in the matching domain file (or create a new `<Name>Api.cs` partial-class file if no existing one fits). Signature: `private Task HandleFooAsync(HttpListenerRequest req, HttpListenerResponse res)`.
3. **Decide whether the endpoint should work without a configured project.** If yes, add the path to `AlwaysAllowed` in [ApiRouter.cs](../../src/Api/ApiRouter.cs). Default: "requires configured context".
4. **Keep handlers thin** — delegate to services in [src/Services/](../../src/Services/).
5. **Read request body** with `await HttpJson.ReadBodyAsync(req)`. Write responses with `await HttpJson.WriteAsync(res, obj)` and errors with `await HttpJson.WriteErrorAsync(res, status, message)`.
6. **Validate paths** from the client through `PathSecurity.Resolve(baseDir, userInput)` — never `Path.Combine` directly with user input. Return 403 when `Resolve` returns null.
7. **Streaming endpoints** (like `/api/events`) must return `true` from the dispatcher; the events path is hard-coded in `DispatchAsync`.

## Required follow-ups when adding/changing endpoints

- **Test:** add a test in [tests/ss14-editor.Tests/ApiRouterTests.cs](../../tests/ss14-editor.Tests/ApiRouterTests.cs) exercising the new handler via the public dispatcher. The `BuildCtx()` helper + `ApiTestServer.Start(...)` pattern starts a real `HttpListener` on an ephemeral port.
- **Frontend:** if the endpoint is consumed from the UI, expose it in [WebUI/js/api.js](../../WebUI/js/api.js). The frontend never calls `fetch` directly outside that file.
- **README:** the [README.md](../../README.md) does not currently enumerate endpoints, but if the new endpoint is user-visible (e.g. CLI exposes it), update the relevant section.

## Things the dispatcher already does — do not duplicate

- JSON content-type is set globally by `DispatchAsync`. Do not set it per handler.
- 404 fallthrough is handled centrally. Do not write per-handler 404s for unknown paths.
- The "no project configured → 503" check is automatic for any path not in `AlwaysAllowed`.

## Security invariants — do not change without discussion

- **No CORS headers.** [EditorServer.cs](../../src/Http/EditorServer.cs) deliberately does NOT send `Access-Control-Allow-Origin: *`. Same-origin policy is the only thing preventing arbitrary websites from issuing `/api/file` POST requests against the user's local listener.
- **Engine prototypes are read-only.** Paths starting with `ProtoIndexService.EnginePrefix` must reject write attempts with 403 (see `HandleFileAsync`).

---
> Source: [TheShuEd/SS14Editor](https://github.com/TheShuEd/SS14Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
