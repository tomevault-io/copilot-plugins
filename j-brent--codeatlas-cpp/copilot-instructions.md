## codeatlas-cpp

> VS Code extension for visualizing C++ codebases as zoomable semantic graphs.

# CodeAtlas C++

VS Code extension for visualizing C++ codebases as zoomable semantic graphs.

## Quick Start

```bash
# Install dependencies
npm install

# Compile extension and webview
npm run compile

# Watch mode (for development)
npm run watch
```

## Development

### Running the Extension

1. Open this folder in VS Code
2. Press F5 to launch the Extension Development Host
3. In the new window, run "CodeAtlas: Open Visualization" from the command palette

### Running the Webview Harness

The webview in a plain browser, no Extension Development Host. Faster loop for anything
webview- or producer-side, and it exercises the real producer adapter, not a hand-built model.

```bash
npm run compile:webview && npm run serve:harness   # http://localhost:5178
```

Renders `webview/harness/blackjack.graph-document.json` — regenerate it after changing
`src/model/toGraphDocument.ts` with `npm run build:harness-fixture`. The page sets
`window.__ATLAS_HARNESS__`, which exposes `window.__ATLAS_RENDERER__` for scripted scene
navigation (`setScope`, `getCytoscape`).

### Project Structure

```
src/                    # Extension code (TypeScript, Node.js)
  extension.ts          # Entry point
  webview/              # WebviewPanel management
  model/                # Semantic model types, builders, GraphDocument adapter
  providers/            # Data providers (clangd, cmake, git)

webview/                # Webview code (TypeScript, browser)
  src/
    main.ts             # Entry point
    graph/              # Cytoscape.js rendering
    diff/               # Git diff animation
  harness/              # Browser harness + checked-in fixture (see above)

shared/ir/              # Graph IR types; generated/ is derived from graph/schema (never edit)
graph/                  # Python package: IR JSON Schema, validator, format converters
tests/                  # Vitest suites (tests/webview/dom/** runs under jsdom)
```

### Build System

- **esbuild** bundles both extension and webview
- Extension: CommonJS, Node.js target, excludes `vscode`
- Webview: ESM, browser target, bundles all dependencies

### Testing

Two runners. Both must pass before committing.

```bash
npm test                  # vitest (extension + webview)
npm run test:coverage     # ...with coverage thresholds enforced (vitest.config.ts)
cd graph && uv run pytest # IR schema + validator suite
```

Typecheck and lint separately — `tsc` covers three projects, including `tsconfig.test.json` for
`tests/` (vitest transpiles without typechecking, so tests are only checked here):

```bash
npx tsc --noEmit -p tsconfig.json && npx tsc --noEmit -p webview/tsconfig.json \
  && npx tsc --noEmit -p tsconfig.test.json
npm run lint
```

`.github/workflows/ci.yml` runs all of the above, and additionally fails if `shared/ir/generated/`
or `webview/harness/` drift from their generators.

### Testing with Real Codebases

The extension works best with C++ projects that have:
- `compile_commands.json` (from CMake with `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON`)
- CMake build system (for target/library detection)

Test against:
- `projects/blackjack/` in this repo (already has CMake)
- FTXUI (~15K LOC, clean modern C++)
- Dear ImGui (~50K LOC, popular and readable)

## Architecture Notes

### Semantic Model

The model is conceptual, not file-based:
- Targets (CMake libraries/executables) at the top
- Classes and functions nested within targets
- Files are metadata, not hierarchy levels

### Cytoscape.js

- Uses compound nodes (nodes containing nodes) for hierarchy
- fcose layout algorithm for force-directed compound graphs
- Canvas rendering for performance

### Extension ↔ Webview Communication

Messages via `postMessage`:
- Extension → Webview: `modelUpdate`, `refresh`, `navigateTo`
- Webview → Extension: `nodeClick`, `exportSvg`, `ready`

## Current Status

Actively being redesigned — not a finished, static project. Current architecture, vision,
and roadmap: [`docs/README.md`](docs/README.md). `.dev-team/` is historical only; do not
treat it as authoritative.

---
> Source: [j-brent/codeatlas-cpp](https://github.com/j-brent/codeatlas-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
