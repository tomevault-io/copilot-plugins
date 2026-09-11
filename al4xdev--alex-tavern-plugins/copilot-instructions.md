## alex-tavern-plugins

> Read `docs/manifest.md`, `docs/sdk.md`, and `docs/hooks.md`; model-backed plugins must also read

# Plugin authoring rules

Read `docs/manifest.md`, `docs/sdk.md`, and `docs/hooks.md`; model-backed plugins must also read
`docs/model-calls.md`. Then query the live core contract through the MCP before selecting extension
points. Use MCP scaffold, validate, test, and pack tools instead of copying an older package.

Plugins are trusted in-process code. Declare every meaningful access for review, but do not mistake
permissions for a sandbox. Prefer transactional filters and namespaced `plugin_state`; use `unsafe`
when the requested feature genuinely needs an unanticipated core object, and document that choice.

Every catalog artifact must be reproducibly packed from the source beside it and pinned by SHA-256.
An Experience contains only ordered plugins, configuration, description, and preview media. It does
not contain scenarios or characters.

The authoring MCP never performs Git operations. Do not commit, push, publish, or modify remotes
without explicit user authorization.

---
> Source: [al4xdev/alex-tavern-plugins](https://github.com/al4xdev/alex-tavern-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
