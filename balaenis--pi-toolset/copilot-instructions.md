## pi-toolset

> Maintain the `pi-toolset` monorepo of Pi extension packages (currently `@balaenis/pi-lsp`, `@balaenis/pi-format`, `@balaenis/pi-agents`). Before acting, identify the deliverable, constraints, and allowed side effects. Choose the most efficient path that satisfies them.

# Instructions

## Goal

Maintain the `pi-toolset` monorepo of Pi extension packages (currently `@balaenis/pi-lsp`, `@balaenis/pi-format`, `@balaenis/pi-agents`). Before acting, identify the deliverable, constraints, and allowed side effects. Choose the most efficient path that satisfies them.

## Success Criteria

- The requested change is complete and in the requested shape
- Validation passes before reporting done
- Missing information, permissions, or blockers are surfaced rather than hidden

## Constraints

- Keep comments to a minimum — let the code speak for itself

### Development

Use `mise` to manage the development environment; `bun` as the package manager; `bunx` for one-off tools.

- `mise run typecheck --package packages/pi-lsp` — TypeScript type check with the TS7 native compiler (tsgo), per package
- `mise run build --package packages/pi-lsp` — Build a package (bun build bundling + postbuild gates)
- `mise run test --package packages/pi-lsp` — Run a package's tests
- `hk check` — eslint + prettier (repo-wide)
- `hk fix` — apply eslint + prettier fixes (repo-wide)

### Documentation

- Update the README with any new or changed usage instructions.
- Add any new or changed configuration options to the README.
- If there are suitable test instruction steps, put them in the README.

## Validation

After a change, run the most relevant check: targeted tests for changed behavior, type checks, lint. If validation cannot run, state why and give the next best check.

## Stop Rules

- Ask before: replacing an entire implementation, destructive or irreversible actions, changing secrets, or acting outside the request scope
- Stop and report when a change requires more context than available

---
> Source: [balaenis/pi-toolset](https://github.com/balaenis/pi-toolset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
