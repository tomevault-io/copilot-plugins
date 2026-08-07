## localpager

> These instructions apply to this repository.

# AGENTS.md

These instructions apply to this repository.

## Slophammer

This repo follows `osolmaz/slophammer` standards. If applying or updating
those standards, start from:

```text
https://raw.githubusercontent.com/osolmaz/slophammer/refs/heads/main/docs/AGENT_ENTRYPOINT.md
```

## Local Checks

Run these before finishing changes:

```sh
gofmt -w cmd internal
./scripts/check.sh
```

Run `slophammer-go check . --execute` when changing quality gates, CI, or build
scripts.

## Go

- Keep package APIs small and explicit.
- Return errors with context.
- Keep interfaces near their consumers.
- Keep domain rules separate from command flags, databases, network calls,
  clocks, and process state.
- Do not add global mutable state for domain behavior.
- Add or update nearest tests when changing behavior.

## Dependencies

Do not add dependencies unless they remove real complexity. Prefer the Go
standard library and existing repo tooling.

---
> Source: [osolmaz/localpager](https://github.com/osolmaz/localpager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
