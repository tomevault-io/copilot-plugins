## browsercode

> - To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.

- To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.
- Prefer automation: execute requested actions without confirmation unless blocked by missing info or safety/irreversibility.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### Schema Definitions (Drizzle)

Use snake_case for field names so column names don't need to be redefined as strings.

```ts
// Good
const table = sqliteTable("session", {
  id: text().primaryKey(),
  project_id: text().notNull(),
  created_at: integer().notNull(),
})

// Bad
const table = sqliteTable("session", {
  id: text("id").primaryKey(),
  projectID: text("project_id").notNull(),
  createdAt: integer("created_at").notNull(),
})
```

## Testing

- Avoid mocks as much as possible
- Test actual implementation, do not duplicate logic into tests
- Tests cannot run from repo root (guard: `do-not-run-tests-from-root`); run from package dirs like `packages/opencode`.

## Type Checking

- Always run `bun typecheck` from package directories (e.g., `packages/opencode`), never `tsc` directly.

---

## BrowserCode-specific notes

This repo is a fork of `anomalyco/opencode` that adds browser-use integration.
The opencode style above still applies to all TS code. A few extras:

### Modification zones

See `UPSTREAM.md` for the canonical version. Short form:

- **Green (add freely):** new files, new packages. `packages/bcode-browser/`
  is the home for BrowserCode-specific code (decisions §1d).
- **Yellow (touch + document):** edits to `packages/opencode/` source go in
  the maintainer-side `memory/browsercode/EXCEPTIONS.md` (kept outside
  this repo with the agent's roadmap/decisions docs) with justification.
  Every Yellow edit is a future merge-conflict candidate, so keep them
  surgical.
- **Red (never touch):** `@opencode-ai/*` package names, `@opencode/...`
  Effect service IDs, `x-opencode-*` wire headers, `OPENCODE_*` env vars,
  third-party provider User-Agents.

### Three-level architecture (decisions §1c)

- **Level 1** — pure additions in `packages/bcode-browser/`. No upstream
  diff. Always preferred.
- **Level 2** — thin adapters in `packages/opencode/src/tool/` that wrap
  Level-1 implementations. Small, mostly schema/context translation.
- **Level 3** — modifications to upstream source. Last resort. Document in
  the maintainer-side `memory/browsercode/EXCEPTIONS.md`. Always evaluate
  whether the change could be upstreamed as an extension point first.

### Vendored harness

`packages/bcode-browser/harness/` is vendored from
`browser-use/browser-harness`. Path-allowlist policy (post upstream PR #229
src-layout reorg):

- `agent-workspace/agent_helpers.py` — editable. Primary BrowserCode
  extension surface.
- `src/browser_harness/*.py` (`daemon.py`, `admin.py`, `helpers.py`,
  `run.py`, `_ipc.py`) — protected. Pull verbatim. If behavior change is
  needed, upstream a PR to `browser-use/browser-harness`.
- `interaction-skills/` — verbatim. Never edit.
- `(agent-workspace/)?domain-skills/` — **excluded** from vendored tree.
  Sync agents skip these paths; see UPSTREAM.md §3 "Excluded paths".

Sync workflow lives in `harness-sync.md`.

### Upstream sync workflow

Pull from `anomalyco/opencode`: see `opencode-sync.md`. Pull from
`browser-use/browser-harness`: see `harness-sync.md`. Both append a row
to `UPSTREAM.md`'s sync log.

When a sync PR is open, **do not land feature work on `main`** — it
creates conflicts the sync agent has to redo. Wait for the sync to merge
first.

### PRs

Use the REST endpoint via `curl`, not `gh pr create` (the project's PAT
allows the REST mutation but not the GraphQL one used by `gh`). Templates
live in `opencode-sync.md` and `harness-sync.md`.

### Filtered typecheck

Root `bun run typecheck` uses a turbo filter limiting to the packages we
ship (`@browser-use/browsercode-core`, `@browser-use/bcode-browser`,
`@opencode-ai/{shared,plugin,sdk}`). This avoids upstream packages we
don't build (e.g. `enterprise`, `web`, `console`). The pre-push hook
runs this filtered command.

---
> Source: [browser-use/browsercode](https://github.com/browser-use/browsercode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
