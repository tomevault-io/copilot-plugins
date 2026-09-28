## hermes-kanban-gantt

> Operating instructions for coding agents (Hermes, Codex, Claude Code, …).

# AGENTS.md — Hermes Kanban Gantt

Operating instructions for coding agents (Hermes, Codex, Claude Code, …).

## Project purpose

A Gantt timeline view plugin for Hermes Desktop: full-page `/kanban-gantt`
route rendering any Hermes kanban board as a timeline of real work
timestamps, with a task detail drawer, filters and bulk operations.
Zero API keys, zero model tokens.

## Architecture map

```
plugin.yaml              unified plugin manifest
__init__.py              no-op register() (capability probe only)
dashboard/manifest.json  dashboard-plugin manifest (api: plugin_api.py)
dashboard/plugin_api.py  BACKEND — FastAPI router: gantt snapshot from the
                         shared kanban SQLite store, task detail, status
                         transitions, comments, bulk ops
desktop/plugin.js        RENDERER — plain ESM, loaded uncompiled; contains a
                         pure Gantt core (GANTT_CORE_SRC template string,
                         extracted at test time) + the React-lite page
install.sh               optional installer (desktop half → desktop-plugins/,
                         backend half → plugins/)
tests/                   node:test (core, sticky label), pytest backend suite,
                         demo server (one port: demo.html + /plugin.js + API)
```

- **Backend owns** all SQLite/kanban data access. Routes live under
  `/api/plugins/kanban-gantt/`.
- **Renderer owns** the visuals. `plugin.js` must stay SELF-CONTAINED (only
  `@hermes/plugin-sdk`, `react`, `react/jsx-runtime` imports — rewritten by
  the desktop loader). The pure timeline logic lives in the
  `GANTT_CORE_SRC` template string so `tests/gantt-core.test.mjs` can
  extract and run the exact shipped source.
- **Persistence**: plugin prefs (`labelW`, `drawerW`, `drawerDocked`,
  `board`, `zoom`, `disabledStatuses`, `baseUrl`) go through `ctx.storage`
  — never localStorage directly.
- **Desktop titlebar**: the board switcher is contributed to
  `titleBar.center` via `<Contribute>` (mounted with the page). The desktop
  shell must render that slot centered and outside window drag regions —
  see NousResearch/hermes-agent#111819.

## Tests

```bash
node --test tests/gantt-core.test.mjs tests/test_sticky.mjs
tests/run_tests.sh        # pytest backend (isolated venv, HERMES_AGENT_HOME)
```

## Gotchas

- Radix `asChild` slots accept a SINGLE element child — arrays throw
  "Primitive.button failed to slot onto its children".
- Electron drag regions ignore `pointer-events: none`; interactive chrome in
  the titlebar band needs an explicit no-drag on its own subtree.
- `plugin.js` is loaded uncompiled: `node --check` + the node:test suite are
  the fastest regression gate before any reinstall.

---
> Source: [e-is/hermes-kanban-gantt](https://github.com/e-is/hermes-kanban-gantt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
