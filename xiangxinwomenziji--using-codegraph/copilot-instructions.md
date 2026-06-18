## using-codegraph

> Use when searching, reading, adding, or modifying code in this workspace. Ensures codegraph MCP tools are consulted first for all code exploration tasks before falling back to Read/Grep/Glob.


# Using codegraph for Code Operations

## The Rule

For any code-related task in this workspace, **start with codegraph MCP tools**. Only fall back to `Read` / `Grep` / `Glob` when codegraph is provably insufficient.

codegraph is a pre-built SQLite index of every symbol, edge, and file. Reads are sub-millisecond. Using it first saves context, avoids re-reading files, and follows call paths grep cannot (callbacks, React re-render, dynamic dispatch).

## When to Use codegraph

- Searching for a symbol, function, class, type, or route
- Asking "how does X work", "where is Y", "what calls Z"
- Tracing call flow between modules
- Impact analysis before refactor
- Finding similar patterns before writing new code
- Surveying an area of the codebase

## Tool Selection by Intent

| Intent | Tool |
|---|---|
| "How does X work?" / architecture / area survey | `codegraph_explore` (PRIMARY — one call usually suffices) |
| Trace flow from X to Y | `codegraph_explore` with both symbol names |
| Find one symbol's location | `codegraph_search` |
| "What calls this?" / "Who imports module X?" | `codegraph_callers` (on an exported symbol of the module) |
| "What does this call?" | `codegraph_callees` |
| "What would changing X break?" | `codegraph_impact` |
| One symbol's full body (or overloaded name) | `codegraph_node` |
| Indexed file tree | `codegraph_files` |
| Index health / debugging index | `codegraph_status` |

Default rule: **`codegraph_explore` first**. It is Read-equivalent — returns verbatim source grouped by file. Most often the only call needed.

### Tool-Specific Quirks (learned the hard way)

- **`codegraph_node` with `file:line` only finds top-level / exported symbols.** A symbol defined inside a function body (e.g. `switchPersona` inside `AuthProvider`) will return "not found". Fall back: `codegraph_search` then `codegraph_explore` with bag-of-names.
- **`codegraph_callers` is incomplete for React hooks.** Calls inside JSX expressions (`<Can permission="...">`) are indirect and not in the call graph. Don't rely on caller count for hook refactors — also grep the page files by hand if needed.
- **`codegraph_files` `maxDepth`** counts path segments, not directory levels. `path: "src/lib"` + `maxDepth: 2` may not behave as expected; prefer `codegraph_search` for locating files.
- **`codegraph_explore` is keyword fuzzy match**, not a true semantic search. Off-topic queries ("the meaning of life") can return noise from identifier names. If the query is clearly not about code, **do not call codegraph** — go straight to `Read` / answer from context.

## When NOT to Use codegraph (Fallback OK)

- codegraph returns no useful result AND you've confirmed it's indexed
- You need the literal, current byte-for-byte content of a file (e.g., binary, generated)
- The file was just created and the indexer hasn't caught up (wait ~1s, retry, then fallback)
- You're writing/editing a file (codegraph is read-only)

When falling back, prefer `Read` over `Grep` when you know the file path. Reserve `Grep` for content patterns across unknown files; reserve `Glob` for filename patterns.

## Red Flags — STOP and Reconsider

You're about to skip codegraph if you think:

- "Just one file, faster to Read directly" — codegraph_explore often returns that file + its callers in one call
- "Grep is faster for this" — codegraph is sub-ms; grep scans disk
- "codegraph might be stale" — index lags ~1s. Only fall back if you need ground truth
- "codegraph is too abstract" — pass symbol names; it returns verbatim source
- "I just need to verify one line" — `codegraph_search` returns line + file, no body needed
- "I'm writing new code, codegraph doesn't apply" — search for similar patterns FIRST
- "The query is too specific" — try `codegraph_explore` with bag of symbol names; it dedups

**All of these mean: try codegraph first. Fall back only after it returns nothing useful.**

## Common Mistakes

| Mistake | Fix |
|---|---|
| Reaching for `Read` immediately | Default to `codegraph_explore` |
| Running `Grep` then `Read` in sequence | One `codegraph_explore` likely replaces both |
| Reading whole files to find one function | `codegraph_node` returns just that function |
| Forgetting to update codegraph intuition | Re-read this skill when adding code — `codegraph_impact` shows what would break |
| Calling many read tools when one explore would do | codegraph IS the search index; don't redo its work |
| Trusting `codegraph_callers` count for hook refactors | Hooks are called via JSX, not in the call graph — verify by hand |
| Calling `codegraph_node` on a function-internal symbol | Returns "not found" for non-top-level; use `codegraph_explore` instead |

## Quick Example

❌ **Without this skill** — burning context:
```text
Read src/lib/auth.tsx
Read src/lib/permissions.ts
Grep "usePermission" src/
Read src/components/rbac/can.tsx
```

✅ **With this skill** — one call:
```text
codegraph_explore("usePermission Can AuthProvider permissions.ts")
```
Returns verbatim source of all four locations, grouped by file, with call paths.

---
> Source: [xiangxinwomenziji/using-codegraph](https://github.com/xiangxinwomenziji/using-codegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
