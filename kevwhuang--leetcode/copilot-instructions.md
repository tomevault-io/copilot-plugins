## leetcode

> LeetCode solutions in JavaScript. Problems without JS support (concurrency, Pandas, etc.) use Java, Python, or SQL.

# CLAUDE.md

LeetCode solutions in JavaScript. Problems without JS support (concurrency, Pandas, etc.) use Java, Python, or SQL.

## Commands

| Command           | Description              |
|-------------------|--------------------------|
| `bun run lint`    | Run ESLint               |
| `bun run gen`     | Generate stub files      |
| `bun test`        | Run all tests            |
| `bun test <file>` | Run a specific test file |

## File Conventions

| Extension | Usage                         |
|-----------|-------------------------------|
| `.js`     | Free JavaScript               |
| `.jsx`    | Free JavaScript (React)       |
| `.mjs`    | Premium JavaScript            |
| `.cjs`    | Free JavaScript (CommonJS)    |
| `.p.cjs`  | Premium JavaScript (CommonJS) |
| `.sql`    | Free database                 |
| `.p.sql`  | Premium database              |
| `.java`   | Java (concurrency)            |
| `.p.java` | Premium Java                  |
| `.py`     | Python (Pandas)               |
| `.sh`     | Shell                         |

## Architecture

```
data-structures/  Custom implementations (Heap, Trie, SegmentTree, etc.)
neetcode/         NeetCode 150 solutions by category
problems/         Solutions by difficulty and ID range
public/           Generator scripts
tests/            Jest tests for neetcode/
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevwhuang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
