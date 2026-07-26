## ogcode

> **Rule:** Before exploring any file, folder, or project structure, you **MUST** use the `codebase_map` tool first.

# Agent Behavioral Instructions

## Mandatory: Use Project Index Before Exploration

**Rule:** Before exploring any file, folder, or project structure, you **MUST** use the `codebase_map` tool first.

This applies to all of the following scenarios:

- **Starting a new task** — Call `codebase_map` (optionally scoped with `subdir`) before reading any source files.
- **Looking for a file** — Use `codebase_map` with an appropriate `subdir` to locate files, instead of guessing paths with `glob` or `grep`.
- **Understanding project structure** — Use `codebase_map` to get the labeled tree of topics and files before diving into code.
- **Exploring a new package/directory** — Call `codebase_map` with `subdir` set to that package/directory.

### Why?

The project index provides topic labels and a structured overview of every indexed file. Using it first ensures:

1. **Faster navigation** — You immediately know which files are relevant without blind `glob`/`grep` searches.
2. **Better context** — Topic labels help you understand what each file contains before reading it.
3. **Fewer mistakes** — You won't miss important files or read irrelevant ones.

### Workflow

```
Task received
  → codebase_map(subdir=...)   ← MANDATORY FIRST STEP
  → Then read specific files
  → Then make changes
```

### When `codebase_map` is not enough

If `codebase_map` doesn't cover what you need (e.g., unindexed files, binary patterns), you may fall back to `glob` and `grep`. But `codebase_map` must always be the **first** exploration step.

### Scoping tip

For large projects, always use the `subdir` parameter to scope `codebase_map` to the relevant directory (e.g., `"internal/tool"`, `"web/src"`). This keeps the response focused and fast.

---
> Source: [prasenjeet-symon/ogcode](https://github.com/prasenjeet-symon/ogcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
