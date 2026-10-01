## gladys

> > Part of the [external integrations living spec](../README.md) — the layout, the editing rules and the cross-repo map are there.

> Part of the [external integrations living spec](../README.md) — the layout, the editing rules and the cross-repo map are there.

# C. Interface specification (v1 contracts)

The files of this folder specify the **v1 core surface**. The contracts added by an integration type or a capability (its manifest field, its endpoints, its WebSocket messages, its SDK methods) are specified in the capability's own file under `../capabilities/` — the tables here only carry what shipped with the core.

General conventions, aligned with the existing code:
- **REST**: JSON exclusively; errors in the standard Gladys format `{ "status": <HTTP code>, "code": "NOT_FOUND" | "UNAUTHORIZED" | "FORBIDDEN" | "BAD_REQUEST" | "UNPROCESSABLE_ENTITY" | ..., "message": "..." }` (produced by `errorMiddleware`).
- **WebSocket**: existing envelope `{ "type": "<namespace.kebab-case>", "payload": { ... } }` (`formatWebsocketMessage`).
- **Dates**: ISO 8601 UTC. **External identifiers**: every integration `external_id` is prefixed `ext:<selector>:` (the server rejects everything else).

---
> Source: [GladysAssistant/Gladys](https://github.com/GladysAssistant/Gladys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
