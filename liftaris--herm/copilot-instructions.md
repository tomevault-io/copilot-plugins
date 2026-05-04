## herm

> - ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.

- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.
- The default branch in this repo is `main`.
- Prefer automation: execute requested actions without confirmation unless blocked by missing info or safety/irreversibility.
- This is a TUI app built with OpenTUI React (`@opentui/react`), NOT React DOM. JSX renders to a terminal.
- Data from `~/.hermes/` carries `source: Source` provenance. Use `<FileLink>` in UI, never hardcode paths.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Prefer single word variable names where possible
- Use Bun APIs when possible, like `Bun.file()`, `bun:sqlite`, `Bun.Glob`
- Bun auto-loads `.env` — don't use dotenv
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream

### Naming

Prefer single word names for variables and functions. Only use multiple words if necessary.

### Naming Enforcement (Read This)

THIS RULE IS MANDATORY FOR AGENT WRITTEN CODE.

- Use single word names by default for new locals, params, and helper functions.
- Multi-word names are allowed only when a single word would be unclear or ambiguous.
- Do not introduce new camelCase compounds when a short single-word alternative is clear.
- Before finishing edits, review touched lines and shorten newly introduced identifiers where possible.
- Good short names to prefer: `pid`, `cfg`, `err`, `opts`, `dir`, `root`, `child`, `state`, `timeout`.
- Examples to avoid unless truly required: `inputPID`, `existingClient`, `connectTimeout`, `workerPath`.

```ts
// Good
const foo = 1;
function journal(dir: string) {}

// Bad
const fooBar = 1;
function prepareJournal(dir: string) {}
```

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json();

// Bad
const journalPath = path.join(dir, "journal.json");
const journal = await Bun.file(journalPath).json();
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a;
obj.b;

// Bad
const { a, b } = obj;
```

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2;

// Bad
let foo;
if (condition) foo = 1;
else foo = 2;
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1;
  return 2;
}

// Bad
function foo() {
  if (condition) return 1;
  else return 2;
}
```

### OpenTUI Gotchas

- `<text>` children must be strings, `<span>`, `<strong>`, `<u>` — never `<box>` or nested `<text>`.
- Mouse events (`onMouseDown`, `onMouseOver`, etc.) work on `<box>` and `<text>`, NOT `<span>`.
- To make inline content clickable, use `<box height={1}>` with handlers, `<text>` inside.
- `useRef()` requires an initial argument — use `useRef<T>(null)`, not `useRef<T>()`.
- `<scrollbox scrollY>` for scrollable content.

## Testing

- Avoid mocks as much as possible
- Test actual implementation, do not duplicate logic into tests
- Run tests with `bun test`

## Type Checking

- Always run `bunx tsc --noEmit` after changes.

---
> Source: [liftaris/herm](https://github.com/liftaris/herm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
