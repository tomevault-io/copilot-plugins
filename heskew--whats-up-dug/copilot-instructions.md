## whats-up-dug

> Interactive terminal data exploration CLI for Harper. Read-only. Full-screen TUI using alternate screen buffer.

# dug

Interactive terminal data exploration CLI for Harper. Read-only. Full-screen TUI using alternate screen buffer.

## Stack

- **TypeScript** + **Bun** (build & runtime)
- **React 18** + **Ink 5** (terminal UI framework)
- **Zod** for API response validation
- **Commander** for CLI argument parsing
- **snooplogg** for debug logging (file-based, opt-in via `SNOOPLOGG` or `DEBUG` env)

## Architecture

```
src/
  index.tsx          Entry point: alt screen buffer, commander args, Ink render
  app.tsx            Root shell: navigation stack, key handling, screen routing, clearScreen on nav
  logger.ts          snooplogg setup — writes to ~/.dug/debug.log when enabled
  relationships.ts   Relationship inference from API metadata + naming conventions
  api/
    client.ts        HarperClient — Operations API over HTTP, Basic Auth, caching, retries
    types.ts         Zod schemas for all Harper API request/response shapes
  hooks/
    use-api.ts       Generic async data fetching hook with loading/error/queryTime
    use-navigation.ts  Push/pop screen stack (connect → dashboard → database → table → record)
    use-terminal-size.ts  Reactive terminal dimensions, computes tablePageSize
  screens/
    connect.tsx      Connection form with saved recent connections (~/.dug/connections.json)
    dashboard.tsx    Database list with fuzzy filter
    database.tsx     Table list for a database with expanded info
    table.tsx        Paginated data grid with overlays: search, sort, columns, query builder, schema info
    record.tsx       JSON tree detail view with FK navigation (Enter follows forward relationships)
    system.tsx       Harper system_information display
  components/
    breadcrumb.tsx   Navigation breadcrumb bar
    data-table.tsx   Data grid with vertical viewport clipping and horizontal column windowing
    json-tree.tsx    Syntax-highlighted JSON renderer with annotations and line selection
    key-hints.tsx    Bottom key hint bar
    query-builder.tsx  Condition builder with attribute/comparator/value editing
tests/
  test-helpers.ts    Shared test factories: makeSchema, makeAttr, mockFetch
  api/               Tests for HarperClient and Zod schemas
  hooks/             Tests for navigation reducer, useApi, useTerminalSize
  screens/           Tests for all screen components
  components/        Tests for DataTable, JsonTree, KeyHints, Breadcrumb, QueryBuilder
```

## Key Patterns

### Navigation
Push/pop stack in `use-navigation.ts`. Screens are React components switched in `app.tsx`. The root `Box` is constrained to `height={terminalRows}` with `overflow="hidden"`, so Ink naturally overwrites all visible lines on navigation — no manual screen clearing needed.

### Overlay Pattern
Table screen uses an `OverlayMode` union type. Overlays have their own `useInput` hooks. A shared `overlayActive` ref prevents the global Escape handler from popping navigation when an overlay is open.

### Viewport Clipping & Column Windowing
`DataTable` and `JsonTree` only render rows/lines visible in the terminal. `useTerminalSize` provides reactive `tablePageSize` that drives both the API fetch limit and the render window. `RESERVED_LINES = 12` accounts for all chrome.

`DataTable` also windows columns horizontally: natural widths (min 8, max 30) are computed from all page data, then columns are packed left-to-right starting from `colStart` until the available width is filled. `h`/`l` keys scroll columns. Row strings are pre-built as plain text to avoid Ink flex layout alignment issues.

### Layout Constraints
The root `Box` in `app.tsx` is constrained to `height={terminalRows}` to prevent Ink from rendering beyond the terminal viewport. The middle content area uses `flexGrow={1} flexShrink={1} overflow="hidden"` and the bottom key hints bar uses `flexShrink={0}` to stay anchored. Key hints truncate via `wrap="truncate"` in narrow terminals.

### Harper Operations API
All requests POST to the instance URL with Basic Auth. Uses `schema` (not `database`) as the field name in request bodies. Empty conditions fall back to `search_by_value` with `*` wildcard (requires limit/offset). `indexed` field can be an object on some Harper versions — Zod uses `z.unknown().transform(v => v === true)`.

### Relationship Inference
`inferRelationships()` checks API metadata first (`.catchall(z.unknown())` passes extra fields), then falls back to naming conventions: `fooId` / `foo_id` → matches against known table names (case-insensitive). Forward refs enable Enter-to-follow in record detail. Reverse refs shown as informational hints.

## Local Data

- `~/.dug/connections.json` — recent URLs and usernames (never passwords)
- `~/.dug/debug.log` — only when `SNOOPLOGG` or `DEBUG` env var is set

## Build

```sh
bun install
npm run build     # → dist/index.js
npm run compile   # → standalone binary
bun test          # → 131 tests across 10 files
```

## Known Issues / TODO

- Minor visual glitches may still occur in very small terminals or with rapid scrolling
- Tests live in `tests/` mirroring the `src/` directory structure. Pure logic is tested directly; components use `ink-testing-library`. Several helpers (`truncate`, `formatCell`, `padCell`, `coerceValue`, `renderJson`, `isNetworkError`, `formatZodError`, navigation `reducer`) are exported for testability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heskew)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/heskew)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
