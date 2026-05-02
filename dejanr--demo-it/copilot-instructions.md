## demo-it

> Read `MIND_MAP.md` before any code change and update it after meaningful changes.

# AGENTS.md

## Mind Map

Read `MIND_MAP.md` before any code change and update it after meaningful changes.
Start with overview nodes `[1-5]`, then follow `[N]` references relevant to the task.

## Architecture Guardrails

- `demo-itd` is the canonical source of run state.
- CLI (`demo-it`) and Neovim plugin are thin clients over daemon protocol.
- Keep transport and state boundaries aligned with `docs/PROTOCOL.md` and `docs/CLI-PLAN.md`.

## Development Environment

Use Nix + devenv + direnv:

```bash
direnv allow
```

To run `devenv` scripts inside the Nix sandbox without entering an interactive shell:

```bash
nix develop --impure --command <script>
# example:
nix develop --impure --command tests
```

Common workflows:

- `devenv up` starts `demo-itd` (with auto-reload via `air`)
- `ci` runs format, lint, tests, and build

## Conventions

- Use [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`).
- Prefer small, focused tests for business logic.
- Keep solutions simple and functional.

## CI Discipline

For this repository, run local CI before committing:

```bash
ci
```

Use targeted commands (`tests`, `lint`, `fmt`) when iterating, then finish with full `ci`.

## Deploy

No deployment workflow is defined in this repository yet.

## Session End

Do not leave a dirty working tree: commit or stash all intentional changes.

---
> Source: [dejanr/demo-it](https://github.com/dejanr/demo-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
