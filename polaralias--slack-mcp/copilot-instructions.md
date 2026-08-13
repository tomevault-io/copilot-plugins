## slack-mcp

> This repository is in a native-runtime hardening phase.

# AGENTS

## Purpose

This repository is in a native-runtime hardening phase.

Agent work should optimise for:

- repository truth
- reproducible verification
- contract preservation
- maintainability of the Python-only FastMCP implementation

## Current reality

- The user-facing server is Python/FastMCP.
- The runtime implementation is now Python-native end to end.
- Historical Go and npm-backed runtime evidence may still appear in dated documents, but they are not supported product paths.

Canonical entry docs:

- [GLOSSARY.md](GLOSSARY.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [docs/DESIGN.md](docs/DESIGN.md)
- [docs/PLANS.md](docs/PLANS.md)
- [docs/SECURITY.md](docs/SECURITY.md)
- [docs/RELIABILITY.md](docs/RELIABILITY.md)

High-signal current-state references:

- [docs/refactor-and-repair-plan.md](docs/refactor-and-repair-plan.md)
- [docs/runtime-validation-2026-05-16.md](docs/runtime-validation-2026-05-16.md)
- [docs/product-specs/auth-model.md](docs/product-specs/auth-model.md)
- [docs/design-docs/auth-principles.md](docs/design-docs/auth-principles.md)

## Working principles

- Treat docs as claims until verified.
- Treat code as intended behaviour until tested.
- Prefer black-box MCP boundary validation over implementation assumptions.
- Do not silently collapse distinctions between:
  - current supported Python-native behaviour
  - historical runtime evidence
  - desired future contract refinements

## Documentation rules

- New decisions should land in canonical docs under `docs/`.
- Legacy docs should not be deleted until their replacements are complete.
- When a legacy doc is contradicted by validation, note that explicitly in canonical docs.
- Configuration and tool-surface claims must identify whether they describe:
  - canonical intended support
  - validated live runtime support
  - legacy or inherited behaviour

## Change rules

- New behaviour should be specified in product- or design-level docs before implementation when the behaviour is user-visible.
- New tests should prefer Python black-box MCP boundary assertions over backend-internal assumptions.
- Do not reintroduce delegated backend patterns or secondary runtime paths without an explicit product decision and matching contract updates.

## Shared Git Workflow

- work from a short-lived branch created from `main`
- do not commit directly to `main`
- use branch names prefixed with `feat/`, `fix/`, `docs/`, `chore/`, `refactor/`, or `test/`
- keep one logical change per branch and pull request
- open a pull request before merging to `main`, including for solo work
- prefer squash merge unless multiple commits carry durable review value
- delete the merged or closed feature branch after the work is finished; never delete `main`
- use tags in `vX.Y.Z` format for releases and do not move published tags

---
> Source: [polaralias/slack-mcp](https://github.com/polaralias/slack-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
