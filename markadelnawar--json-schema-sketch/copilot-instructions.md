## json-schema-sketch

> A zero-dependency TypeScript library that does two things:

# json-schema-sketch

## What this is

A zero-dependency TypeScript library that does two things:

1. **Schema inference** — takes any JSON value and produces a compact type-tree (`SchemaNode`), then renders it as minimal text. Output like `array(25) of {id: number, name: string}` — not standard JSON Schema.

2. **Path resolution** — resolves dot/bracket notation paths (`data.users[*].id`) against JSON values. Supports wildcards, array indexing, and nested navigation with structured error messages.

## Architecture

```
src/
  index.ts          — barrel re-export (8 symbols: SchemaNode, InferOptions, RenderOptions, inferSchema, renderSchema, ResolveResult, ResolveError, resolvePath)
  types.ts          — SchemaNode, InferOptions, RenderOptions interfaces
  inference.ts      — inferSchema() + helpers (sampling, merging, optional keys, union deduplication)
  rendering.ts      — renderSchema() + helpers (inline/multiline formatting, variant counts)
  path-resolver.ts  — resolvePath(), ResolveResult, ResolveError
test/
  inference.test.ts   — 16 tests (primitives, arrays, optional keys, depth, exhaustive mode, unions, dedup, varying sizes, mixed types)
  rendering.test.ts   — 26 tests (all rendering paths: inline, multi-line, dot-paths, optionals, unions, variant counts, varying sizes)
  path-resolver.test.ts — 13 tests (keys, indices, wildcards, errors)
```

Source files have zero runtime dependencies — only `import type` between internal modules.

## Key design decisions

- **Two inference modes**: `sampled` (default) picks 3 array elements (first, middle, last) for speed. `exhaustive` checks every element to catch mixed types and rare keys.
- **Array sampling**: In sampled mode, `pickSampleIndices()` selects max 3 elements. But ALL elements are scanned for key discovery (to catch optional keys). Donor elements provide types for keys missing from the sample.
- **Optional key detection**: Keys that appear in only some array elements are marked `optional: true`.
- **Varying array sizes**: When merging arrays, if their counts differ → `varyingSizes: true` instead of a misleading count from the first array.
- **Union deduplication**: `mergeSchemas` groups schemas by type, merges each group via `mergeSameType`, and produces a union with `variantCount` per variant.
- **Array merging in `mergeSameType`**: Arrays are merged by combining their item schemas and checking if counts vary — not just returning the first one.
- **Compact text rendering**: Inline rendering for objects under 80 chars (120 for array items), multi-line with dot-path expansion for larger ones.
- **Path wildcards**: `[*]` maps over array elements with a `maxItems` cap (default 5). Returns `totalItems` so the caller knows the full size.
- **Structured errors**: `resolvePath` errors include available keys so LLMs can self-correct typos.
- **Default maxDepth**: 10 (bumped from 6 to handle real-world CMS responses with 7+ levels of nesting).

## Build & test

```bash
npm run build    # tsc → dist/
npm test         # build + node --test dist/test/*.test.js
```

Uses Node's built-in test runner (`node:test` + `node:assert/strict`). No test framework dependency. 55 tests total.

## Publishing

- `files` in package.json is set to `["dist/src"]` — only compiled source ships, not tests.
- Run `npm pack --dry-run` to verify before publishing.
- ESM only (`"type": "module"`, NodeNext module resolution).

---
> Source: [markadelnawar/json-schema-sketch](https://github.com/markadelnawar/json-schema-sketch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
