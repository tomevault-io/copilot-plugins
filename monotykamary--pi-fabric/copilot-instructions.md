## pi-fabric

> Always finish a change with a fresh build before handing it back:

# AGENTS.md

## Golden rule: build when done

Always finish a change with a fresh build before handing it back:

```sh
bun run build
```

Pi loads and publishes the compiled bundle in `dist/` — not `src/`. Tests run
against `src/`, so green tests alone are not enough: without a build the
change is invisible in the TUI and unpublished. Rebuild so the user can
verify immediately.

## Startup: budget the transitive graph, not just the entrypoint

A warm jiti filesystem cache avoids transpilation, **not V8 compilation** in a
new Pi process. A small entrypoint can still import megabytes through barrels,
shared chunks, schema builders, or supposedly cheap catalog metadata. Native
ESM may bypass jiti's virtual host modules and compile another dependency copy.
Do not assume that marking a package external makes startup free.

Keep registration and idle lifecycle hooks cheap. Load optional engines,
parsers, serializers, and UI catalogs at actual first use, not in
`session_start`, `resources_discover`, an immediately invoked async function,
or a zero-delay "background" import. Use `import type` for type-only edges.
For genuinely synchronous render APIs, memoized first-use `createRequire`
loads of optional **third-party** dependencies preserve behavior. Do not use
this escape hatch for Pi host APIs: lazy UI must use the supplied host interfaces.
Keep package-local dynamic entry paths stable across installed generations.

When touching startup, run `bun run build` (the artifact check guards the whole
static closure and optional dependencies), `bun run assert:lazy-graph`, and
cold-import/idle plus first-use tests. Review shared chunks too: code splitting
can pull a lazy module back into the eager graph. Do not raise a graph budget
just to make a regression pass.

Measure in fresh processes, with the host preloaded and warm transpile caches:

```sh
bun run benchmark:startup . ../pi-fovea ../pi-contour
```

This reports import and registration separately; it is not a full-session boot
measurement. Same-process repeated imports measure the module cache, not boot.
Compare before/after on the same machine. The probe interleaves extension order
and reports CPU time as well as wall time; inspect sample spread on a busy host.
Keep CI assertions structural rather than enforcing flaky millisecond thresholds.

## Checks are incremental, never a full sweep

Never run the whole test suite. Verify the files a change touches:

```sh
bun run check:fast                           # typecheck + affected tests
bun run test:changed                         # dirty src/tests files vs HEAD
bun run test:affected                        # CI mode: $PI_TEST_BASE vs HEAD
bun run test:smoke                           # curated cross-platform floor
bun run test:related -- src/ui/settings.ts   # tests importing given files
bunx vitest run tests/agent-manager.test.ts  # one suite, by path
bun run typecheck                            # whole-program types, seconds
```

`scripts/test-affected.mjs` backs the first three. It feeds changed `src/` and
non-test `tests/` files to `vitest related`, runs changed `tests/**/*.test.ts`
directly, and drops deleted paths. That selection avoids the vitest
`--changed` pitfall where a dirty `package.json` forces the entire suite to
run. An unreadable git state fails loudly. Nothing falls back to the full
suite, and there is no `bun run check` or `bun run test` script: those
aggregates are gone on purpose.

CI selects by range: the workflow passes the pull-request base or the replaced
push tip as `PI_TEST_BASE` and compares it to `HEAD` as a three-dot diff. A
docs-only push selects nothing. The curated `test:smoke` floor then runs on
both Ubuntu and Windows, which is what keeps a platform-specific break from
riding in on a change the selection did not cover.

`bun run lint:dead` (knip dead-code audit) and `bun run assert:lazy-graph`
(host-free lazy UI graph) are on-demand audits. Run them deliberately when a
change is about dead exports or the lazy startup graph, never as a routine
gate.

Publishing runs `prepack`, which is `typecheck && build` (seconds), never the
suite. After a version bump the artifact is verified through
`dist-tags` and `time["<version>"]` in the packument.

## Package manager

This repo is bun-managed (`bun.lock`; scripts invoke `bun run` internally).
Do not use pnpm/npm — pnpm's pre-run dependency check fails against the bun
lockfile.

## Commits

Use conventional commits (commitlint): `feat(scope): ...`, `fix(scope): ...`,
`chore(release): <version>` for version bumps.

---
> Source: [monotykamary/pi-fabric](https://github.com/monotykamary/pi-fabric) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
