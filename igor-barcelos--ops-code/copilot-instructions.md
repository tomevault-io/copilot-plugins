## ops-code

> **Keep CLAUDE.md up to date.** Whenever features are added, changed, or removed, update this file to reflect the current state. This includes new files, changed data flows, removed commands, and updated message types.

# ops-code

## Maintaining This File

**Keep CLAUDE.md up to date.** Whenever features are added, changed, or removed, update this file to reflect the current state. This includes new files, changed data flows, removed commands, and updated message types.

## Coding Guidelines

1. **Simple is better than complicated.** This is a core principle. When in doubt, choose the simpler approach.
2. **Clean architecture with clearly identified inputs and outputs.** Every module must have an obvious entry point, a clear input, and a clear output. No hidden state, no surprise side effects.
3. **Small parts that do one specific thing.** Prefer many small, focused functions/files over one large file that does everything. Each piece should be independently understandable.
4. **If the code seems too complicated, it probably is.** Stop, step back, and find a simpler solution before continuing.
5. **Types are mandatory.** In TypeScript, `any` is forbidden — every variable, parameter, and return value must be explicitly typed. In Python, type hints must be used consistently. The JSON schema shared between Python and TypeScript must have matching type definitions on both sides.
---

## How this project works

**ops-code** is a VS Code extension that runs openseespy models.

**Flow:**

1. **Trigger** — User runs `ops-code.run` on a `.py` script (right-click or active editor).
2. **Intercept** (`python/interceptor.py`) — A `ModelInterceptor` monkey-patches openseespy calls to capture nodes, elements, supports, and loads — without running the FEM solver. Stops early via `InterceptorStop`.
3. **Runner** (`python/runner.py`) — Executes the user script with the interceptor injected, suppresses stdout, and emits a single JSON blob to stdout.
4. **Panel** (`src/panel.ts`) — Opens a VS Code Webview panel, spawns the Python process, and passes the JSON to the viewer. A file watcher re-runs on every save.
5. **Viewer** (`media/viewer.ts` → compiled `viewer.js`) — Three.js-based 3D renderer that draws nodes, elements, supports, and nodal loads from the JSON.
6. **Analysis** (optional) — User clicks "Run Analysis" in the viewer; `python/analysis_runner.py` runs the full OpenSeesPy model and returns displacements/reactions/forces, overlaid on the view.
7. **Tools** (optional) — `python/tools_runner.py` + `src/tool_manager.ts` discover and run tool scripts in the model's directory, displaying tabular output in the panel.
8. **Screenshot** — `ops-code.screenshot` command triggers the webview to export a PNG, saved next to the script.

**Key data contract:** `ModelData` / `AnalysisRunnerOutput` (defined in `src/types.ts`) is the shared JSON schema between Python and TypeScript.

---
> Source: [igor-barcelos/ops-code](https://github.com/igor-barcelos/ops-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
