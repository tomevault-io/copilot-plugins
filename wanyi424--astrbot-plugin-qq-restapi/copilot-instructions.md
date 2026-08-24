## astrbot-plugin-qq-restapi

> `main.py` is the plugin entrypoint and registers commands plus platform adapters. Keep public integration points in `public_api.py`; other modules should prefer that surface over deep imports. Core runtime logic lives in `runtime/` (event parsing, sending, token management, template loading), platform wiring lives in `adapters/`, and persistence lives in `db/` (`models.py`, `repository.py`, `service.py`). Stateless QQ API helpers remain in `core/qq/`. Shared templates belong in `templates/` with `templates/registry.yaml`. `wanbot/` is an optional private extension area; avoid coupling public code to it unless the feature is explicitly private.

# Repository Guidelines

## Project Structure & Module Organization
`main.py` is the plugin entrypoint and registers commands plus platform adapters. Keep public integration points in `public_api.py`; other modules should prefer that surface over deep imports. Core runtime logic lives in `runtime/` (event parsing, sending, token management, template loading), platform wiring lives in `adapters/`, and persistence lives in `db/` (`models.py`, `repository.py`, `service.py`). Stateless QQ API helpers remain in `core/qq/`. Shared templates belong in `templates/` with `templates/registry.yaml`. `wanbot/` is an optional private extension area; avoid coupling public code to it unless the feature is explicitly private.

## Modification Scope and Framework Boundary (Highest-Priority Constraint)
This repository contains only the QQ REST API plugin adapter. **Never modify any AstrBot framework source code or files.** Every code, configuration, documentation, and resource change must remain strictly inside the current plugin directory, `AstrBot/data/plugins/qq_restapi/`. Do not edit framework modules, shared components, startup logic, dependency files, sibling plugins, or any other path outside this plugin, and do not indirectly overwrite framework code through copied files, patch scripts, generated artifacts, or similar workarounds.

If analysis or implementation reveals that a requirement cannot be implemented safely and completely without changing the AstrBot framework, stop before making such a change. Do not silently expand the task scope or modify the framework first. Explain to the user which framework location would need to change, why the change appears necessary, its expected impact and risks, and any plugin-only alternatives. Discuss the situation thoroughly and obtain repeated, explicit confirmation from the user. Unless and until the user explicitly revises the task scope and authorizes framework-level work, treat all framework code as read-only and the current plugin directory as the only writable project scope.

## Build, Test, and Development Commands
This plugin is loaded by AstrBot rather than built as a standalone package.

- `python3 -m compileall .` checks Python syntax across the plugin.
- `find . -name '*.py' -exec python3 -c "import ast; ast.parse(open('{}').read())" \\;` performs a lightweight AST parse pass.
- `python3 -c "import json; json.load(open('_conf_schema.json')); print('OK')"` validates the config schema file.

For functional verification, install the repo under `<AstrBot>/data/plugins/qq_restapi` and restart AstrBot to exercise the WebSocket or Webhook adapter.

## Coding Style & Naming Conventions
Use 4-space indentation, type hints where practical, `snake_case` for modules/functions, and `PascalCase` for classes. Match the existing Python style in `main.py` and `runtime/`. Keep adapter code focused on transport concerns; business logic belongs in `runtime/` or `db/`. When adding plugin config, update both `[_conf_schema.json](_conf_schema.json)` and `_PLUGIN_CONFIG_KEYS` in `runtime/context.py`.

## Testing Guidelines
There is no dedicated `tests/` directory yet. Treat syntax checks plus AstrBot smoke testing as the minimum bar. If you add behavior with branching logic, include a focused validation path in docs or a reproducible command example. Prefer testing both adapter modes when touching event parsing, token flow, or reply helpers.

## Commit & Pull Request Guidelines
Git history is minimal (`Initial open-source release`), so use short imperative subjects such as `Add webhook signature validation`. Keep commits scoped to one change. PRs should explain the user-visible effect, note any config/schema updates, link related issues, and include screenshots or log snippets when changing message rendering, templates, or webhook behavior.

## Security & Configuration Tips
Never commit real `appid`, `secret`, webhook UUIDs, or private backend URLs. Keep secrets in AstrBot configuration, and document new config keys in `metadata.yaml` when they are exposed to users.

---
> Source: [Wanyi424/astrbot_plugin_qq_restapi](https://github.com/Wanyi424/astrbot_plugin_qq_restapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
