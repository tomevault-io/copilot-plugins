## ascii-mermaid

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Neovim plugin that renders Mermaid diagram code blocks in Markdown files as inline ASCII/Unicode art using virtual text (extmarks). The rendering engine is a pure TypeScript library (zero dependencies) that runs via Node.js, invoked asynchronously from Lua.

## Build

```bash
cd ts && npm run build    # runs tsup, outputs to ts/dist/
```

No install step needed — there are no runtime dependencies. Dev dependencies (tsup, typescript) are only needed for building.

The Neovim plugin loads the built `ts/dist/index.js` directly via `node --input-type=module`.

## Architecture

### Two-layer design

**TypeScript engine** (`ts/src/`) — Parses Mermaid text and renders it to ASCII/Unicode strings. Exported API: `renderMermaidAscii(text, options)` from `ts/src/ascii/index.ts`. This is synchronous, pure TypeScript.

**Neovim Lua plugin** (`lua/ascii-mermaid/`, `plugin/`) — Detects mermaid code blocks in markdown buffers, shells out to Node.js to run the TS engine, and displays results as virtual text below each block.

### TypeScript rendering pipeline

For **flowcharts and state diagrams** (the original pipeline):
1. `parser.ts` — Regex-based line-by-line parser → `MermaidGraph` (nodes, edges, subgraphs)
2. `ascii/converter.ts` — Converts `MermaidGraph` → `AsciiGraph` (internal grid-based representation)
3. `ascii/grid.ts` → `createMapping()` — Places nodes on logical grid, computes column/row sizes, runs A* pathfinding for edges, converts to drawing coordinates
4. `ascii/draw.ts` → `drawGraph()` — Renders boxes, lines, arrows, corners, subgraph borders onto a 2D character canvas in layered order
5. `ascii/canvas.ts` — Column-major 2D text array (`canvas[x][y]`), with Unicode junction merging

For **sequence, class, ER, Gantt, pie, and timeline diagrams**: Each has its own parser (`sequence/parser.ts`, `class/parser.ts`, `er/parser.ts`, `gantt/parser.ts`, `pie/parser.ts`, `timeline/parser.ts`) and renderer (`ascii/sequence.ts`, `ascii/class-diagram.ts`, `ascii/er-diagram.ts`, `ascii/gantt.ts`, `ascii/pie-chart.ts`, `ascii/timeline.ts`) with layout strategies suited to each diagram type. They bypass the grid/pathfinder pipeline.

### Key coordinate systems

- **Grid coordinates** — Logical grid where each node occupies a 3x3 block. Spacing between nodes is 4 grid units.
- **Drawing coordinates** — Character-level positions on the canvas, derived by summing column widths and row heights.

### Lua plugin modules

- `init.lua` — Setup, user commands (`:MermaidRender`, `:MermaidRenderAll`, `:MermaidClear`, `:MermaidStyle`, `:MermaidMode`), autocmds with debouncing
- `detect.lua` — Finds mermaid code blocks via tree-sitter with regex fallback
- `render.lua` — Async Node.js invocation via `vim.system()`, passes JSON on stdin
- `display.lua` — Manages extmarks, content hashing for dedup, toggle state

### Rendering modes

`useAscii: true` uses `+`, `-`, `|`, `>` characters. `useAscii: false` (default) uses Unicode box-drawing characters (`┌`, `─`, `│`, `►`).

### Display modes

- **inline** — Rendered diagram appears as virtual text below the code block
- **replace** — Diagram overlays the source code; cursor reveals source
- **hybrid** — Replace for short blocks, inline for longer ones
- **readonly** — Overlay that never reveals source

## Supported Diagram Types

- Flowcharts (`graph TD`, `flowchart LR`, all directions including BT via canvas flip)
- State diagrams (`stateDiagram-v2`, including composite states)
- Sequence diagrams (`sequenceDiagram`)
- Class diagrams (`classDiagram`)
- ER diagrams (`erDiagram`)
- Gantt charts (`gantt`, sections, task bars, dependencies, status tags)
- Pie charts (`pie`, horizontal proportional bars with percentages)
- Timeline diagrams (`timeline`, vertical layout with period boxes and event trees)

## Testing

```bash
make test          # run all tests (build + TS unit tests + Neovim integration tests)
make test-ts       # TypeScript unit tests only (node:test)
make test-nvim     # Neovim headless integration tests only
make build         # build only
```

**TypeScript unit tests** (`ts/test/*.test.js`) — one file per diagram type, tests `renderMermaidAscii()` directly with `node:test`. Run a single test file: `cd ts && node --test test/flowchart.test.js`

**Neovim integration tests** (`test/nvim/test_*.lua`) — one file per diagram type, runs `nvim --headless` to test the full pipeline (detect blocks → render via Node.js → verify extmarks/virtual text). Shared helpers in `test/nvim/helpers.lua`. Run a single test: `nvim --headless --noplugin -u NONE --cmd "set rtp^=$(pwd)" -l test/nvim/test_flowchart.lua`

**Manual testing** — open `test.md` in Neovim with the plugin loaded.

## Rules

Before writing any code:

1. **Research codebase patterns first.** Read the relevant existing files to understand naming conventions, error handling style, comment style, and structural patterns. Match what's already there — don't introduce new conventions.
2. **Add test coverage for rendering pipeline changes.** Any edit to `ts/src/` that touches parsing, layout, drawing, or canvas logic must include a corresponding test case in `ts/test/`. Render the affected diagram type before and after to verify no existing cases break. Run `make test` to confirm.
3. **Never use emojis.** This includes common marker emojis like checkmarks and crosses. Use plain symbols instead:
   - `[x]` / `[PASS]` instead of a checkmark emoji
   - `[!]` / `[FAIL]` instead of a cross emoji
   - `--` or `*` for bullets, `=>` for arrows, etc.

Additional guidelines:

- Keep the TypeScript engine zero-dependency. Do not add npm runtime dependencies.
- Lua code uses LuaCATS annotations (`---@param`, `---@return`). Follow the same style for new functions.
- The canvas is column-major (`canvas[x][y]`). Do not confuse this with row-major indexing.
- When adding a new diagram type, it needs: a parser in `ts/src/<type>/`, a renderer in `ts/src/ascii/`, a TS test in `ts/test/`, and a Neovim integration test in `test/nvim/`.
- See `docs/` for detailed architecture documentation.

---
> Source: [kais-radwan/ascii-mermaid](https://github.com/kais-radwan/ascii-mermaid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
