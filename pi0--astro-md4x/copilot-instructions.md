## astro-md4x

> Keep AGENTS.md updated with project status

Keep AGENTS.md updated with project status

# astromd4x

Drop-in replacement for `@astrojs/markdown-remark` powered by [md4x](https://github.com/pi0/md4x). Single runtime dep.

## Source files

- `src/index.ts` — Main exports, `createMarkdownProcessor`, config defaults
- `src/types.ts` — API-compatible types (all Astro types: `MarkdownHeading`, `ShikiConfig`, plugin types, etc.)
- `src/_frontmatter.ts` — `parseFrontmatter`, `extractFrontmatter`, `isFrontmatterValid` using md4x `parseMeta`
- `src/_slug.ts` — GitHub-slugger compatible slug generation with dedup

## Pipeline

1. `md4x.init()` — one-time WASM/NAPI init (cached via `_initialized` flag)
2. `md4x.parseMeta(content)` — extracts frontmatter + `headings[]{level, text}`
3. `md4x.renderToHtml(content)` — renders to HTML (strips frontmatter, GFM built-in)
4. `injectHeadingIds()` — post-processes HTML to add `id` attrs via regex

## Key behaviors

- md4x strips frontmatter automatically in `renderToHtml` — Astro's original does NOT (expects caller to strip). Our processor handles both transparently.
- Slugger creates github-slugger compatible slugs with deduplication per render call.
- `markdownConfigDefaults` and `syntaxHighlightDefaults` exported for Astro config schema compatibility.

## Testing

- `test/index.test.ts` — Unit tests for processor, frontmatter, slugging (18 tests)
- `test/bench.test.ts` — Validation tests ensuring parity with `@astrojs/markdown-remark` (7 tests)
- `test/bench.bench.ts` — Vitest bench comparing both processors (4 scenarios)
- `test/_fixtures.ts` — Shared markdown fixtures (simple, withFrontmatter, complex)

Run: `pnpm vitest run test/index.test.ts test/bench.test.ts`
Bench: `pnpm vitest bench test/bench.bench.ts`

## Workspace

- `pnpm-workspace.yaml` links root + `playground/`
- Root `package.json` has `pnpm.overrides` to alias `@astrojs/markdown-remark` → `workspace:astromd4x@*`
- `astro-markdown-remark` dev dep = aliased `npm:@astrojs/markdown-remark` (bypasses override for benchmarks)
- Must run `pnpm --filter astromd4x build` before playground build

## Reference

`_ref/` — sparse clone of `withastro/astro` `packages/markdown/remark` for API reference.

---
> Source: [pi0/astro-md4x](https://github.com/pi0/astro-md4x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
