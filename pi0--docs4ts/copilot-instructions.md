## docs4ts

> CLI tool and library that extracts JSDoc documentation from TypeScript/JavaScript source files and generates formatted Markdown.

# docs4ts

CLI tool and library that extracts JSDoc documentation from TypeScript/JavaScript source files and generates formatted Markdown.

## Project Structure

```
src/
├── index.ts       # Public API exports
├── cli.ts         # CLI entry point (`docs4ts <file> [-o out]`)
├── parser.ts      # JSDoc extraction & parsing (core logic)
├── loader.ts      # Module graph traversal & re-export resolution
└── markdown.ts    # Markdown rendering
test/
├── parser.test.ts   # Parser tests (~170 assertions)
└── markdown.test.ts # Markdown rendering tests
.agents/
└── OXPARSER.md    # oxc-parser API reference
```

## Architecture

```
CLI / API
  → loader.ts: loadJSDocs(file)
    → Traverse module graph (follows re-exports)
    → For each file: parser.ts: extractJSDocs(source)
      → oxc-parser: parseSync → AST + comments[]
      → Binary search index on comment positions
      → Two-pass visitor:
        1. Mark exported declaration ranges
        2. Collect declarations + match JSDoc by position
      → Return JSDocEntry[]
  → markdown.ts: renderJSDocsMarkdown(entries)
    → Format as Markdown sections
```

## Key Data Types

- **`JSDocEntry`**: `{ kind, name, exported, description?, tags[], signature? }`
- **`JSDocTag`**: `{ tag, name?, type?, description? }`
- Kinds: `function`, `class`, `interface`, `type`, `enum`, `variable`, `method`, `property`

## Public API (`src/index.ts`)

- `extractJSDocs(source, options?)` — Parse single file
- `parseJSDoc(comment)` — Parse JSDoc comment text
- `renderJSDocsMarkdown(entries)` — Render entries to Markdown
- `jsdocsToMarkdown(file)` — All-in-one: file → Markdown
- `loadJSDocs(file)` — Load with re-export traversal

## Scripts

| Command        | Description                               |
| -------------- | ----------------------------------------- |
| `pnpm dev`     | Vitest watch mode                         |
| `pnpm test`    | Lint + typecheck + tests with coverage    |
| `pnpm build`   | Bundle with obuild                        |
| `pnpm fmt`     | automd + oxlint --fix + oxfmt             |
| `pnpm docs4ts` | Run CLI from source (`node ./src/cli.ts`) |

## Dependencies

- **Runtime:** `oxc-parser` — Fast Rust-based TS/JS parser
- **Build:** `obuild` — Bundler (ESM output to `dist/`)
- **Test:** `vitest` with v8 coverage
- **Lint:** `oxlint` + `oxfmt`
- **Types:** `tsgo` for typechecking

## Key Implementation Details

- Comments are **not** attached to AST nodes by oxc-parser; matched by byte position (see `.agents/OXPARSER.md`)
- Binary search for efficient JSDoc-to-declaration matching
- Re-export resolution: follows `export ... from` statements, resolves relative paths, tries `.ts/.tsx/.js/.jsx/.mts/.mjs` extensions
- Signature extraction: raw source text up to first `{` (functions) or first line (types)

@.agents/OXPARSER.md

---
> Source: [pi0/docs4ts](https://github.com/pi0/docs4ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
