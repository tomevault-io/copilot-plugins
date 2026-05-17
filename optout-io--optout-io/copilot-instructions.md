## optout-io

> This file is loaded automatically by Claude Code for any agent working in this workspace. Read this before making changes to any component.

# CLAUDE.md — optout-io workspace

This file is loaded automatically by Claude Code for any agent working in this workspace. Read this before making changes to any component.

## What this system does

optout-io automates privacy opt-out requests (GDPR, CCPA, DPPA) across hundreds of data brokers. Users sign up, the system submits and tracks erasure requests on their behalf.

## System documentation

**Start here:** [`system-designs/ARCHITECTURE.md`](./system-designs/ARCHITECTURE.md)

- Component map, data flows, key invariants
- For *why* decisions were made: [`system-designs/decisions/`](./system-designs/decisions/)
- For feature design docs: [`system-designs/designs/`](./system-designs/designs/)

## Component repos and roles

| Repo | Language | Role |
|---|---|---|
| `data-erasure-wf/` | Go | Core orchestration: Temporal workflows, NATS consumer, gRPC API |
| `lake/` | Go | Event sink: NATS → PostgreSQL, gRPC query API |
| `abscond/` | Go | Admin dashboard: SSE real-time UI, calls lake gRPC |
| `webform-playwright/` | TypeScript | Browser automation: Playwright + Temporal worker |
| `contracts/` | Protobuf | Shared schemas published as Go + TS packages |
| `go-common/` | Go | Shared infra: logging, OTel, URN utils |

## Cross-repo rules

- **contracts/ is the API surface.** Services communicate via protobuf contracts, not shared structs. Any new cross-service message needs a contracts change first.
- **go-common/ is domain-agnostic.** Never add business logic or domain imports there.
- **Each service owns its DB.** No cross-domain database reads.
- **Breaking protobuf changes are forbidden.** Add new fields; never remove or renumber.

## Local development

Each component has its own `CLAUDE.md` with build/test/run commands. Check there first.

Shared infrastructure (Temporal, NATS, PostgreSQL) is started via docker-compose in `data-erasure-wf/`:
```
cd data-erasure-wf
make docker-compose
make local-bootstrap
```

## Working on a feature

1. Check `system-designs/ARCHITECTURE.md` to understand which components are affected.
2. Create a design doc in `system-designs/designs/<feature-name>/` before writing code.
3. Each component gets its own PR. Reference the design doc in each PR description.
4. Update `system-designs/ARCHITECTURE.md` if the feature changes the system structure.

---
> Source: [optout-io/optout-io](https://github.com/optout-io/optout-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
