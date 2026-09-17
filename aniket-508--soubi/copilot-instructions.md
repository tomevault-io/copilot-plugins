## soubi

> This file orients an agent (or human) working **on the Soubi codebase itself**. (For the _example plugin's_ doctrine, see `examples/diff-river/AGENTS.md`.)

# Soubi — repository guide for coding agents

This file orients an agent (or human) working **on the Soubi codebase itself**. (For the _example plugin's_ doctrine, see `examples/diff-river/AGENTS.md`.)

## What this repo is

Soubi compiles one file-based plugin definition into the native artifact each agent harness expects. The repository has one public package plus documentation and examples; the CLI runs from TypeScript source via `tsx`.

## Layout

```
packages/
  soubi/       the only package
    src/
      core/      zero-dependency portable types, capabilities, and emission helpers
      discover/  source manifest discovery, loading, normalization, and diagnostics
      compiler/  adapter orchestration, output invariants, and target exclusion
      runtime/   define* / file() authoring helpers
      adapters/  production emitters for all twenty-two harnesses
      cli/       argument parsing, shared project services, and command modules
      install/   manifest ownership, planning, filesystem safety, and transactions
    test/
      unit/        pure discovery, compiler, and CLI contracts
      adapters/    built-in harness conformance
      integration/ filesystem and marketplace boundaries
      scenarios/   initializer and transactional installation
examples/
  diff-river/  the canonical example plugin (alp-river-shaped)
docs/          fumadocs (Next.js) documentation site
```

## Golden rules

1. **`packages/soubi/src/core` is zero-dependency.** Do not add runtime deps to it. The frontmatter parser is hand-rolled on purpose.
2. **The capability map is the single source of truth.** Teach Soubi about a harness change by editing `packages/soubi/src/core/capabilities.ts`, not by special-casing inside an adapter.
3. **Every feature × harness pair must resolve to a tier** (`support` / `degrade` / `drop` / `unsupported-hard`). Never silently emit a hollow artifact.
4. **One harness opting out never blocks the rest.** Only `unsupported-hard` failures exclude a target, and only that target.
5. **`packages/soubi` is the only package.** Keep internal boundaries as source folders and expose user-facing APIs through `soubi` subpaths.

## Common commands

```bash
pnpm install
pnpm exec tsc --build packages/soubi/tsconfig.json   # typecheck the Soubi package
pnpm test                                           # exercise all twenty-two harnesses
pnpm soubi build --cwd examples/diff-river          # build the example
pnpm soubi check --cwd examples/diff-river          # report all problems
pnpm docs:dev                                        # run the docs site
```

## Adding a harness adapter

See `docs/content/docs/extending.mdx` (or the rendered `/docs/extending`). Declare capabilities in core, implement `emit()` under `packages/soubi/src/adapters/<id>`, then add one descriptor to `HARNESS_DEFINITIONS` in `packages/soubi/src/adapters.ts`.

## Before you finish

- `pnpm exec tsc --build packages/soubi/tsconfig.json` must pass.
- `pnpm test` must pass.
- `pnpm soubi check --cwd examples/diff-river --harness claude,codex,opencode` should report `3 built` with no failures.
- Add a release entry for any change to a publishable Soubi package by running `pnpm tegami` (writes a markdown file under `.tegami/`). See `.tegami/README.md` for the format.

---
> Source: [Aniket-508/soubi](https://github.com/Aniket-508/soubi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
