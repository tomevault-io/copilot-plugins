## delegated-execution-protocol

> This repository is the protocol truth source for delegated execution.

# CLAUDE.md

This repository is the protocol truth source for delegated execution.

## Start Here

Read in this order before changing behavior:

1. `README.md`
2. `docs/current/guides/protocol-pre-split-boundary.md`
3. `docs/current/spec/architecture.md`
4. `docs/current/spec/platform-api-v0.1.md`
5. `docs/current/guides/integration-playbook.md`
6. `docs/current/guides/release-process.md`

## Repository Boundary

This repository owns:

- `@delexec/contracts`
- schemas and validation helpers
- error registry and status enums
- signing and canonicalization rules
- templates and protocol truth-source documentation

This repository does not own:

- caller or responder runtime implementation
- end-user CLI UX
- Docker or compose deployment

## Development Rules

- Change this repository first when request/result semantics, compatibility rules, templates, or schema validation change.
- Keep implementation-specific runtime logic out of this repository.
- Treat docs and packaged assets as part of the protocol surface. Update them together with code.
- Any new externally visible protocol behavior must be reflected in `docs/current/`.

## Validation

Run after meaningful changes:

```bash
npm install
npm test
```

## Release Rule

Release `@delexec/contracts` before updating client or platform repositories to consume new protocol behavior.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hejiajiudeeyu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
