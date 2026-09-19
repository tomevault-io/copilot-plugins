## agentic-identity-broker

> > **IMPORTANT:** Use retrieval-led reasoning. Read the relevant source and documents before you decide. Do not infer project behavior from general knowledge.

# Agentic Identity Broker — Agent Context

> **IMPORTANT:** Use retrieval-led reasoning. Read the relevant source and documents before you decide. Do not infer project behavior from general knowledge.

## How to Use This File

Use this routing document to find repository areas. Before you change an area, read its nearest `AGENTS.md`. Use the index for detailed documents.

## Docs Index (Retrieval Targets)

For compact scanning, use pipe-delimited entries. Before you implement in a domain, read the relevant file.

### Architecture & Decisions

`ARCHITECTURE.md` | Source of truth for system design
`adrs/NNN-*.md` | Binding ADRs — read before implementation (see the ADR index that follows)
`.specify/memory/constitution.md` | Binding constitution — 13 principles governing all work

### Section-Specific Agent Context

`internal/AGENTS.md` | Backend hexagonal architecture, builder pattern, testing conventions
`internal/domain/AGENTS.md` | Domain layer rules, zero-infra imports, data models, error types
`internal/domain/id/AGENTS.md` | Strongly typed entity IDs, code generation, type catalogue
`internal/ports/AGENTS.md` | Port interface catalogue, ISP rules, error conventions
`internal/adapters/AGENTS.md` | Adapter map, cross-adapter ban, storage/encryption/HTTP details
`internal/extproc/AGENTS.md` | Standalone ExtProc token exchange service, config, architecture
`web/AGENTS.md` | React SPA, design system, API client, testing
`infra/AGENTS.md` | AWS CDK encryption stack, IRSA, environment parameterization
`tests/AGENTS.md` | E2E + integration test suite overview
`tests/e2e/AGENTS.md` | Ginkgo E2E rules, fixtures, dual-server pattern, anti-patterns
`tests/e2e/frontend/AGENTS.md` | Playwright browser tests, page objects
`tests/integration/AGENTS.md` | Self-contained and infrastructure-backed integration test guidance

### Runtime Entry Points

`cmd/agentic-identity-broker/` | Broker CLI, configuration, and dual-server startup
`cmd/extproc-token-exchange/` | ExtProc CLI, telemetry bridge, and gRPC startup

### API Contracts

`api/enduser/openapi.yaml` | End-user API: consent, OAuth2, sessions, and approvals
`api/admin/openapi.yaml` | Admin API: agents, services, resources, and permission sets

### Specs (Feature Requirements)

`specs/NNN-<name>/spec.md` | Feature requirements + acceptance scenarios
`specs/NNN-<name>/plan.md` | Implementation approach
`specs/NNN-<name>/tasks.md` | Trackable task list

### Encryption Deep Reference

`.claude/skills/aws-crypto-go/SKILL.md` | AWS Encryption SDK Go — patterns, rules
`.claude/skills/aws-crypto-go/reference.md` | API reference
`.claude/skills/aws-crypto-go/examples.md` | Code examples

### Design System (Frontend)

`web/src/design-system/docs/INDEX.md` | Complete design system documentation index
`web/src/design-system/docs/COMMON_MISTAKES.md` | Read before you work on a styled component
`web/src/design-system/docs/DECISION_TREES.md` | Which component to use when

### Operations & Guides

`docs/ENCRYPTION_INTEGRATION_GUIDE.md` | Encryption integration guide
`docs/STORAGE_EXTENSION_GUIDE.md` | Adding new storage entities
`docs/STORAGE_TROUBLESHOOTING.md` | Storage debugging
`docs/configuration.md` | Configuration reference
`docs/deployment/` | Deployment guides (Kubernetes, IRSA)
`docs/operations/` | Operations runbooks
`examples/config/` | Example configuration files
`charts/agentic-identity-broker/` | Helm deployment contract
`migrations/` | PostgreSQL schema migrations
`mocks/` | Standalone test services and OAuth2 clients

## Constitution (Binding) — 13 Principles

Treat `.specify/memory/constitution.md` as **BINDING**. Apply these principles:

1. **Security-First** — Fail closed. Do not use bypasses. Never treat signature validation as optional.
2. **ADRs are Binding** — Treat ADRs as binding. Use `ARCHITECTURE.md` as the source of truth. A deviation requires a superseding ADR.
3. **Library-First Security** — Do not use custom crypto. Use Go `crypto/*`, `golang.org/x/crypto`, or AWS Encryption SDK.
4. **OpenAPI Transparency** — Define APIs in `api/{enduser,admin}/openapi.yaml` before implementation.
5. **Domain-Driven Design** — Enforce ubiquitous language. Add new concepts to the `ARCHITECTURE.md` glossary.
6. **Hexagonal Architecture** — Arrange dependencies as domain → ports (interfaces) → adapters. Never reverse this order. Prevent these violations:
   - **Port bypass**: Route handlers through domain services to ports. Do not route a handler to a port directly.
   - **Anemic domain**: Make services enforce invariants. Do not only proxy port calls.
   - **Domain logic leakage**: Put conditional logic beyond input parsing in domain services, not handlers.
   - **Domain packaging**: Make each new `domain/X/` a genuinely independent bounded context.
7. **Configuration-Driven** — Get all config through `internal/ports/config.go`. Do not load config ad hoc.
8. **TDD** — Use red-green-refactor. Write tests first. Make each test fail before implementation.
9. **Persistence Consistency** — Define ISP repos in `internal/ports/storage.go`. Use sqlx for PostgreSQL. Provide both in-memory and postgres adapters.
10. **API-First** — Design APIs before implementation. Get stakeholder agreement before changes.
11. **Design System Compliance** — Use Refined Trust Architecture and WCAG 2.1 AA. Use semantic tokens in `web/src/design-system/`.
12. **DI via Builder** — Put all wiring in `internal/app/builder.go`. Do not instantiate services in routing.
13. **E2E Acceptance Tests** — Use a 1:1 specification-to-test mapping in `tests/e2e/`. Test a complete user journey. Do not test one endpoint segment.

## Monorepo Structure

| Section | Technology | AGENTS.md |
|---|---|---|
| `internal/` | Go 1.26.8 — hexagonal (domain→ports→adapters→app) | `internal/AGENTS.md` |
| `web/` | React 19 + TypeScript + Vite 7 + Tailwind 4 | `web/AGENTS.md` |
| `infra/cdk/` | AWS CDK (Go) — KMS, DynamoDB, IAM | `infra/AGENTS.md` |
| `tests/` | Ginkgo/Gomega (e2e), Go testing (integration) | `tests/AGENTS.md` |
| `internal/extproc/` | Standalone gRPC ExtProc service | `internal/extproc/AGENTS.md` |

**Import rules**: Do not let the domain import adapters or app. Define contracts in ports.
Let driven adapters import ports and domain. HTTP routing receives handlers from app.
It can use routing middleware and configuration. Do not let adapters import each other.

## ADR Decision Index

Read relevant ADRs before implementation. Treat accepted ADRs as authoritative. Treat a new ADR in the same PR only as a proposal.

| ADR | File | Decision |
|---|---|---|
| 002 | `adrs/002-configuration-libraries.md` | Viper + Cobra + godotenv config |
| 003 | `adrs/003-chi-framework.md` | chi v5 HTTP router |
| 004 | `adrs/004-dual-server-isolation.md` | errgroup dual-server :8000/:14000 |
| 004 | `adrs/004-storage-layer-architecture.md` | Hexagonal storage, ISP repos, sqlx |
| 005 | `adrs/005-spa-serving-pattern.md` | Embedded SPA with history API fallback |
| 006 | `adrs/006-frontend-stack.md` | React 19 + Vite + Tailwind + Vitest |
| 007 | `adrs/007-e2e-testing-with-ginkgo.md` | Ginkgo v2 BDD E2E tests |
| 008 | `adrs/008-encryption-context-optimization.md` | EncryptionContext = `service_id` only |
| 008 | `adrs/008-token-exchange-jwks-adapter-pattern.md` | JWKS adapter with jwx jwk.Cache |
| 009 | `adrs/009-cel-for-authorization-policies.md` | cel-go for authorization + JWT claims |
| 009 | `adrs/009-envelope-encryption-design.md` | Three-layer envelope: KEK→BranchKey→DEK |
| 009 | `adrs/009-separate-migration-docker-image.md` | Separate migration Docker image |
| 010 | `adrs/010-cdk-encryption-infrastructure.md` | AWS CDK for encryption IaC |
| 011 | `adrs/011-extproc-standalone-binary.md` | ExtProc as standalone binary |
| 011 | `adrs/011-opentelemetry-provider-pattern.md` | OpenTelemetry provider pattern |
| 012 | `adrs/012-encryption-layer-separation.md` | Encryption layer separation |
| 012 | `adrs/012-extproc-in-memory-token-cache.md` | ExtProc in-memory token cache |
| 013 | `adrs/013-strongly-typed-entity-ids.md` | Typed entity IDs (`XxxID`) in `domain/id/` |
| 014 | `adrs/014-long-poll-listen-notify.md` | Long-poll approval sync via PostgreSQL LISTEN/NOTIFY |
| 014 | `adrs/014-oauth2-server-mode.md` | OAuth2 authorization server mode |
| 015 | `adrs/015-cimd-fetcher-architecture.md` | CIMD fetcher architecture |
| 016 | `adrs/016-authorization-session-anti-spoofing.md` | Authorization session anti-spoofing |
| 017 | `adrs/017-optional-agent-client-id.md` | Optional/nullable Agent client_id |
| 018 | `adrs/018-approval-endpoint-auth-boundaries.md` | Endpoint-specific approval authentication boundaries |
| 027 | `adrs/027-extproc-telemetry-shared-dependency.md` | ExtProc telemetry shared dependency |
| 028 | `adrs/028-opa-extproc-authorization.md` | OPA authorization in ExtProc via embedded SDK |
| 029 | `adrs/029-token-exchange-client-assertion-trust-anchor.md` | Dedicated client-assertion trust anchor for token exchange |
| 030 | `adrs/030-normalize-protected-resources.md` | Normalized, globally unique protected-resource child records |
| 030 | `adrs/030-vendor-neutral-oidc-trust.md` | Vendor-neutral CDK OIDC trust configuration |
| 035 | `adrs/035-root-mounted-spa.md` | Root-mounted SPA (first-class routes; /consent unmounted) |
| 032 | `adrs/032-impersonation-requires-user-delegation.md` | User delegation required for OAuth2 impersonation |

## Domain Glossary

| Term | Definition |
|---|---|
| **Agent** | AI agent with optional client ID and client metadata document URLs. |
| **ThirdpartyOAuth2Service** | External OAuth2 provider with credentials and scopes. |
| **UserGrant** | User delegation of permission sets to an agent. One per user-agent pair. |
| **AuthorizationSessionClaims** | JWE-sealed authorization context with a 10-minute TTL. |
| **UserSession** | Authenticated OAuth2 session with encrypted tokens. |
| **Principal** | Authenticated user ID from `X-Remote-User`. |
| **Secret** | Value object with exclusive plaintext or encrypted state. |
| **ServiceRequirement** | Agent requirement for a mandatory or optional service. |
| **BranchKey** | DynamoDB-cached key between a KMS KEK and an operation DEK. |
| **EncryptionContext** | AAD with exactly one subject key: `service_id` or `kid`. Never store secrets. |
| **ResourceURI** | Normalized protected-resource URI for RFC 8693 token exchange. |
| **CEL Expression** | Policy for privileged-client authorization and JWT claim extraction. |
| **ToolApproval** | Human-in-the-loop authorization for an agent tool call with pending, approved, or denied status. |

## Development Workflow

Use `just` as the command runner. Run `just --list` for the full listing.

| Command | Purpose |
|---|---|
| `just check` | **Static checks**: fmt → vet → lint |
| `just build` | Build Go binary → `./bin/agentic-identity-broker` |
| `just test` | Fast Go/package tests (no E2E or integration suites) |
| `just verify` | Full verification gate with E2E as the final guard layer |
| `just build-all` | Backend + frontend build |
| `just test-e2e` | All backend, ExtProc, and frontend E2E suites |
| `just test-integration` | Default self-contained integration suites |
| `just test-integration-infra` | Infra-backed integration suites (Docker/Podman required) |
| `just test-integration-all` | Both integration layers |
| `just cdk-test` | CDK unit tests |

**Local validation:** Do not rely only on editor diagnostics. After Go, test, or infrastructure changes, run `just check`. Then run the smallest matching test command.

## Code Style

Use these rules for all monorepo code (Go, TypeScript, CDK):

- **Minimize comments** — Write a comment only for context that code alone cannot convey.
- **Never restate code** — Do not write comments that add no information.
- **No debug artifacts** — Do not leave debug logging, commented-out code, TODO stubs, or future-implementation references.
- **Match surrounding style** — Keep naming, error handling, and file structure consistent with adjacent code.
- **Explicit over clever** — Make intent clear without tracing abstractions.
- **Extend, do not duplicate** — Use an existing mechanism (for example, JWE for ephemeral state). Do not introduce a competing one.

- **No test-only production APIs** — Put test helpers in `_test.go` or `tests/`. Assert through production APIs.

## Database Migrations

Use `NNN_description.{up,down}.sql` for migrations in `migrations/`. For large tables, use `CREATE INDEX CONCURRENTLY` with the no-transaction directive.

## Active Technologies

Go 1.26.8 | React 19 + TypeScript + Vite 7 + Tailwind 4 | chi v5 | sqlx + pgx v5 | Ginkgo/Gomega | Viper/Cobra | CVA | OpenTelemetry (`otelhttp`, `otelslog`) | `envoyproxy/go-control-plane` | PostgreSQL (prod) + in-memory (dev/test)

---
> Source: [zalando-incubator/agentic-identity-broker](https://github.com/zalando-incubator/agentic-identity-broker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
