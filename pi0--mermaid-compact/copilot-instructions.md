## mermaid-compact

> Keep AGENTS.md updated with project status

Keep AGENTS.md updated with project status

# Project status

`mermaid-compact` prebundles [mermaid](https://mermaid.js.org) into a single
self-contained ESM file so consumers avoid its ~200-chunk lazy dependency graph.
Concretely: a trivial `import mermaid; initialize()` app built with Vite emits
**95 JS chunks**; the same app importing `mermaid-compact` emits **1**.

## How it works

- `src/index.ts` re-exports mermaid's public API (`export * from "mermaid"` +
  default). mermaid, rolldown, jsdom and vite are **devDependencies** — they're
  bundled in or only used for building/testing, never shipped as runtime deps.
- `build.config.ts` drives obuild:
  - `rolldownOutput` hook sets `codeSplitting: false` so all dynamic diagram
    imports inline into one `dist/index.mjs` (no chunk waterfall).
  - The `excludeDiagrams` rolldown plugin stubs heavy packages
    (`EXCLUDED_PACKAGES`: the **cytoscape** family + **katex**) and the
    **architecture** diagram definition (`EXCLUDED_DIAGRAM_DEFS`). Stubs are a
    callable throwing Proxy so both function- and object-shaped usage fails
    loudly. Edit these lists to change scope.
  - rolldown-plugin-dts bundles mermaid's types into a self-contained
    `dist/index.d.mts`.
- Output: `dist/index.mjs` ≈ 2.7 MB min / ~700 kB gzip, all diagram types except
  architecture, and no KaTeX math labels.

## Testing

`test/index.test.ts` runs under vitest's jsdom environment (`test/setup.ts`
polyfills SVG `getBBox`/`getComputedTextLength`). It imports the **built**
`dist/index.mjs` (building first if absent), renders each core diagram type, and
asserts architecture diagrams throw. `pnpm test` = lint + typecheck + vitest.

## Size levers (from the rolldown bundle-analyzer)

- cytoscape/architecture: dropped (−640 kB). ✅
- katex (math labels): dropped (−240 kB). ✅
- `@mermaid-js/parser` (Langium): shared by pie/gitGraph/info/… — **cannot** be
  dropped without losing common diagrams. (Its "1.3 MB" in the analyzer is
  unminified source; it collapses heavily.)
- swimlanes (ELK flowchart layout), venn.js: small / shared — not dropped.

## Tree-shaking (investigated, not adopted)

Consumer-side tree-shaking is **not achievable** with mermaid: `render`/`parse`
(and even `registerExternalDiagrams`) call `addDiagrams()` unconditionally, which
eagerly references every built-in diagram's loader, and diagram type is detected
at runtime from the input text. So a downstream bundler can't drop unused diagrams;
`sideEffects: false` doesn't help. mermaid also ships no stable per-diagram runtime
modules (only hash-named chunks + `.d.ts`).

The achievable equivalent is a build-time **allowlist** (invert `EXCLUDED_*` into an
include list, stubbing every non-listed diagram definition). Deliberately **not**
adopted: (1) any non-listed diagram then throws — bad for undocs, which renders
arbitrary user diagrams; (2) new upstream diagram types would default to off;
(3) tighter coupling to internal chunk names; (4) small payoff — ~1.5 MB is shared
engine (Langium parser, d3, dagre, dompurify, marked) every diagram needs, so even
a flowchart-only build floors at ~1.55 MB (vs 2.33 MB current). We keep the
exclusion model instead.

---
> Source: [pi0/mermaid-compact](https://github.com/pi0/mermaid-compact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
