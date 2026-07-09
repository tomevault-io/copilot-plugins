## h3-rules

> Keep AGENTS.md updated with project status.

# AGENTS.md

Keep AGENTS.md updated with project status.

`h3-rules`: declarative route rules (redirect, proxy, headers, cache, basicAuth, cors) for H3 v2. Runtime matcher (rou3) + build-time compiler subpath (`h3-rules/compiler`).

Invariants live in `.agents/` — read them before changing matching, merging, or the compiler:

- [`.agents/SECURITY.md`](.agents/SECURITY.md) — security invariants (do not weaken).
- [`.agents/INVARIANTS.md`](.agents/INVARIANTS.md) — behavioral invariants and cross-file contracts to keep in sync.

## Commands

- `pnpm test` — lint + typecheck + vitest w/ coverage (run before committing)
- `pnpm fmt` — fix lint/format (oxlint + oxfmt); CI fails on unformatted files
- `pnpm build` — obuild; entries: `src/index.ts`, `src/compiler.ts`
- `pnpm bench` — matcher throughput (standalone [mitata](https://github.com/evanwashere/mitata), no vitest: `node --expose-gc bench/match.mjs`); `--expose-gc` lets mitata GC between benchmarks for stabler samples. mitata is a devDependency for this bench only.
- `pnpm bench:size` — per-built-in-rule consumer bundle sizes (esbuild, `bench/bundle-size.mjs`), measuring compiled-mode per-rule tree-shaking. esbuild is a devDependency for this bench only. Keep `bench/rules.ts` in sync when adding a built-in rule.

## Constraints

- Runtime code must stay runtime-agnostic: Web APIs only, no `node:*` imports, no side effects, no unstorage/srvx dependencies. Runtime deps are exactly `rou3`, `ufo`, `ocache`; `h3` is a **peer** (pinned `^2.0.1-rc.23` — minimum version exporting `resolveDotSegments`, required by `canonicalPath`).
- Keep `rou3/compiler` imports confined to `src/compiler.ts` so they stay out of runtime bundles.
- The export surface must stay **tree-shakeable**, including namespace member access (`import * as rules from "h3-rules"; rules.xyz`): no module-level side effects in runtime code, `/* @__PURE__ */` on module-scope instantiations (e.g. the shared `cache` instance). `test/treeshake.test.ts` pins this with esbuild metafile assertions (named + namespace imports, compiled codegen output, a positive rou3-present control), including a forced-`sideEffects: true` variant — the package-level `"sideEffects": false` hint would otherwise let bundlers drop unused modules regardless of content, masking a stray module-scope side effect or a lost `@__PURE__`.
- ocache storage is process-global (`setStorage`) — the `cache.storage` matcher option mutates it; per-instance isolation is only possible via a `defineCachedHandler` replacement.
- `src/index.ts` is the explicit export surface — no barrel re-export sprawl; keep modules small and single-purpose. Modules with no public exports live in `src/internal/` (`key.ts`, `premerge.ts`, `scope.ts`); everything exported from `src/index.ts` must be README-documented API or a type reachable from one. (`scope.ts` was public pre-release; re-export `canonicalPath`/`isPathInScope` if a consumer needs the hardened scope check for custom handlers.)

---
> Source: [h3js/h3-rules](https://github.com/h3js/h3-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
