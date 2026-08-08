## trueai-harness

> This file defines how humans and coding agents must design, implement, test, and

# Engineering and Architecture Guide

## Purpose and Scope

This file defines how humans and coding agents must design, implement, test, and
review this repository. It applies to the entire repository unless a more
specific `AGENTS.md` exists in a subdirectory.

The architectural source of truth is
[`SAP_AI_Agent_Platform_OpenHands_Implementation_Plan.md`](SAP_AI_Agent_Platform_OpenHands_Implementation_Plan.md).
Read the relevant sections of that document before making architectural,
security, runtime, workflow, SAP integration, or data-model changes.

This guide turns that product and architecture plan into day-to-day engineering
rules. It must not be used to weaken a boundary or control from the reference
plan. When requirements conflict or an important decision is not documented,
stop, state the conflict, and create or propose an Architecture Decision Record
(ADR).

## Default Engineering Rules

- Use **Python 3.13** for backend and platform code.
- Use **TypeScript in strict mode** only for frontend code.
- Choose the simplest solution that fully meets the current requirement.
- Do not add speculative abstractions, frameworks, services, or dependencies.
- Keep functions and modules small, readable, typed, and easy to test.
- Give every module one clear responsibility.
- Separate business rules from APIs, databases, frameworks, and vendor SDKs.
- Depend on interfaces at system boundaries; keep implementations replaceable.
- Do not let one component perform validation, authorization, execution,
  approval, and publication as a single responsibility.
- Reuse existing project patterns before creating new ones.
- Add tests for new behavior and bug fixes.
- Run Ruff, Pyright, and pytest before considering work complete.

Use these dependency boundaries:

```text
API and external adapters -> application use cases -> domain
```

The domain contains business rules. The application layer coordinates use
cases. Ports define required interfaces. Adapters connect databases, APIs, SAP,
OpenHands, Restate, and other external systems. The API layer only handles
transport concerns.

## Product Direction

Build a configuration-driven SAP AI agent platform in which:

- OpenHands is the execution kernel for agent reasoning, conversations, tools,
  context management, and isolated workspace operations.
- The product control plane owns tenant configuration, identity, authorization,
  run records, approvals, artifacts, and audit indexing.
- A durable workflow engine owns long-running phase state, retries, timers,
  human waits, cancellation, recovery, and coordination of external effects.
- The Runtime Manifest Builder deterministically resolves and freezes all
  approved runtime configuration before execution begins.
- The SAP MCP Gateway is the only route from agent execution to SAP systems.
- SAP remains the system of record for business data.
- Product behavior varies through versioned configuration, process packs,
  skills, capability providers, policies, and templates—not through
  customer-specific branches in the shared runtime.

Complete one production-quality vertical slice before expanding to additional
use cases. Prefer an end-to-end path with real validation over several partially
implemented components.

## Technology and Language Baseline

Use **Python 3.13** as the primary backend language. The control plane, Runtime
Manifest Builder, configuration resolver, durable workflow handlers, Runtime
Manager, OpenHands integration, SAP MCP Gateway, capability providers, artifact
service, and audit service should be implemented in Python unless an approved
ADR establishes a concrete reason to use another language.

Use **TypeScript in strict mode** for the web console and browser-side code.
Do not add a second backend implementation in TypeScript merely to share types
with the frontend. Exchange versioned OpenAPI, JSON Schema, and event contracts
and generate language-specific clients or models where practical.

Use YAML or JSON only for declarative, versioned configuration. Configuration
must select registered implementations and must not become an embedded
programming language.

The initial Python toolchain should use:

- `uv` for Python version, dependency, lockfile, and workspace management.
- Pydantic models for validated external and persisted contracts.
- FastAPI for REST, SSE, health, and administration endpoints.
- The Restate Python SDK for durable workflow handlers.
- `pytest` for tests, Ruff for formatting and linting, and Pyright for static
  type checking.
- Full type annotations for application, domain, port, and public adapter code.

Pin the exact Python version and all dependency versions in repository-managed
files and runtime images. Keep framework-specific types at adapter boundaries.

## Simplicity, Modularity, and Separation of Duties

Write the simplest code that correctly satisfies the current requirement and
preserves the architecture. Simple means easy to read, test, change, and
operate—not compressed, clever, or incomplete.

### Keep code simple

- Prefer clear, direct code over clever patterns or unnecessary indirection.
- Implement only the current requirement. Do not build speculative extension
  points, generic frameworks, or features that have no concrete use.
- Start with a function or small module. Introduce a class only when it owns
  meaningful state, behavior, or a clear interface.
- Create an abstraction only when it expresses a domain concept, protects an
  architectural boundary, or removes proven repetition.
- Prefer standard-library and existing project capabilities before adding a
  dependency.
- Keep control flow shallow. Use early validation and explicit returns instead
  of deeply nested conditions.
- Use descriptive names and small cohesive units. A reader should understand a
  module's purpose without tracing unrelated code.
- Avoid hidden behavior, magic configuration, global mutable state, import-time
  work, and implicit dependency lookup.
- Delete obsolete code instead of retaining unused compatibility layers. Keep
  compatibility only when a documented contract requires it.
- Optimize only after measurement identifies a real bottleneck.

### Enforce modularity

Each module or component must have one primary responsibility and one clear
owner. Keep business decisions separate from transport, storage, framework, and
vendor code.

- `domain` defines business concepts and invariants. It performs no network,
  database, filesystem, framework, or OpenHands operations.
- `application` coordinates use cases through declared ports. It does not know
  which database, workflow engine, SAP protocol, or web framework is used.
- `ports` define the narrow capabilities required by the application.
- `adapters` implement ports for MongoDB, Restate, OpenHands, SAP, object
  storage, identity, HTTP, and other external systems.
- `api` validates and translates transport requests and responses. It does not
  contain business rules or access databases directly.

Dependencies flow inward:

```text
API / adapters -> application -> domain
```

The domain never imports application, adapter, API, framework, or vendor types.
Application code may import domain and ports, but not concrete adapters.

### Segregate duties

- The component that requests an action must not grant its own permission.
  Policy and authorization decisions remain independent.
- The component that executes agent reasoning must not own tenant truth,
  workflow truth, credentials, artifact approval, or audit truth.
- The workflow coordinates phases; it does not implement agent reasoning or SAP
  protocol details.
- The SAP MCP Gateway authorizes and routes capabilities; capability providers
  implement SAP-specific behavior.
- The artifact service stores and versions outputs; workspaces only prepare
  temporary files.
- Audit recording is separate from operational logging and cannot be controlled
  by the code being audited.
- Validation, execution, approval, and publication are separate steps. Do not
  collapse them into one handler.

Do not use modularity as a reason to create unnecessary services. Begin with
well-separated modules in one deployable application when that is sufficient.
Split a module into an independent service only when isolation, scaling,
security, ownership, or deployment requirements justify the operational cost.

### Boundary rules

- A module accesses another module only through its public interface.
- Do not import private implementation details across component boundaries.
- Do not read or write another component's database collections directly.
- Do not create circular dependencies.
- Shared packages contain stable contracts and small domain value objects, not
  service-specific business logic.
- Cross-component calls use typed, versioned contracts and return typed errors.
- Keep transactions within one owning component. Coordinate multi-component
  work through the durable workflow.

## Non-Negotiable Architecture Invariants

Every implementation must preserve these invariants:

1. **No customer hardcoding.** Do not embed tenant names, SAP endpoints,
   credentials, fixed model names, customer templates, process lists, or tool
   assignments in runtime code.
2. **Immutable execution contract.** A run starts only from a validated,
   immutable Runtime Manifest. Every referenced component has an identifier,
   version, and content hash. Secrets appear only as secure references.
3. **Deterministic configuration.** Configuration inheritance and precedence are
   explicit, testable, reproducible, and independent of map iteration order or
   mutable global state.
4. **Capability-based SAP access.** Agents request typed business capabilities.
   Never expose generic SQL, arbitrary RFC/OData, arbitrary ABAP execution,
   unrestricted shell access, or raw customer endpoints.
5. **Independent authorization.** Prompts, model output, plugins, and MCP results
   never grant permission. Authorization is deterministic, enforced at the
   gateway or service boundary, and recorded.
6. **Tenant isolation by construction.** Tenant identity comes from verified
   identity claims, not user-editable input. All reads, writes, cache keys,
   events, artifacts, traces, and queries are tenant-scoped.
7. **Durable external effects.** OpenHands conversation persistence is not a
   workflow transaction. SAP operations and other external side effects use
   durable phase boundaries, stable idempotency keys, and outcome reconciliation
   before retry.
8. **Evidence before assertion.** Publishable statements, findings, and test
   outcomes link to evidence. Unsupported information is labeled as an
   assumption, conflict, or open question.
9. **Separate durable artifacts from workspaces.** A workspace is temporary
   working memory. Important evidence, checkpoints, and outputs are exported to
   durable services before cleanup.
10. **Audit is independent and mandatory.** Configuration, policy, workflow,
    tool, approval, and artifact events go to an append-only audit boundary.
    Configuration cannot disable or silently bypass audit.
11. **Least privilege.** Each run receives only the capabilities, data, network
    access, filesystem scope, resources, and time required by its manifest.
12. **Stable product contracts.** Product APIs and events do not expose
    OpenHands-internal schemas directly. Translate them through owned,
    versioned contracts.

## Logical Architecture

Keep dependencies flowing inward toward stable domain contracts:

```text
Experience/API
      |
      v
Control Plane --> Configuration Resolver --> Runtime Manifest Builder
      |                                      |
      v                                      v
Durable Workflow --------------------> Runtime Manager --> OpenHands
      |                                      |
      |                                      v
      +------------------------------> SAP MCP Gateway --> SAP providers
      |
      +--> Artifact Service / Audit Service / Telemetry
```

### Component ownership

| Component | Owns | Must not own |
| --- | --- | --- |
| Experience layer | Request capture, status, input, approvals, result access | Policy enforcement or direct runtime/SAP access |
| Control plane | Tenants, users, configuration, runs, approvals, artifact catalogue, governance | Agent reasoning or direct SAP execution |
| Configuration resolver | Layered configuration loading and deterministic merging | Runtime execution or secret values |
| Runtime Manifest Builder | Compatibility checks, policy validation, version resolution, manifest freezing | Agent execution or credential storage |
| Durable workflow | Major phases, waits, retries, recovery, cancellation, idempotency coordination | Fine-grained model reasoning |
| Runtime Manager | OpenHands provisioning, conversation lifecycle, normalized runtime events | Tenant truth, workflow truth, or policy ownership |
| OpenHands runtime | Agent loop, approved tools/skills, context, workspace operations | Business configuration, billing, audit truth, or unrestricted secrets |
| SAP MCP Gateway | Capability authorization, system/provider resolution, credential injection, redaction, SAP audit | Prompting or document generation |
| Artifact service | Durable artifact metadata, content, versions, access, retention | Temporary reasoning state |
| Audit service | Immutable security and business event record | Debug traces as the only evidence |

Do not bypass a component to save implementation time. If an early vertical
slice deploys several logical components together, preserve their interfaces and
ownership boundaries in code so they can be separated later.

## Recommended Repository Structure

Use this structure as the repository grows. Do not create empty directories
speculatively; add them with the first real implementation.

```text
apps/
  api/                         # External REST/SSE entry point
  web/                         # Operator and end-user console
services/
  control-plane/
  configuration-resolver/
  runtime-manager/
  workflow-worker/
  sap-mcp-gateway/
  artifact-service/
  audit-service/
packages/
  contracts/                   # Versioned API, event, manifest, capability schemas
  domain/                      # Shared domain value objects; no infrastructure code
  observability/               # Correlation, metrics, tracing, redaction helpers
config/
  platform-defaults/
  use-cases/
  processes/
  agent-profiles/
  workflows/
  skills/
  capabilities/
  policies/
  outputs/
deployments/
  local/
  kubernetes/
docs/
  adr/
  architecture/
  runbooks/
tests/
  integration/
  contract/
  security/
  reliability/
  end-to-end/
```

Inside a service, prefer ports-and-adapters boundaries:

```text
src/
  domain/          # Entities, value objects, invariants
  application/     # Use cases and orchestration
  ports/           # Interfaces required by the application
  adapters/        # MongoDB, workflow, OpenHands, SAP, object store, HTTP
  api/             # Transport handlers and serialization
tests/
```

Domain and application code must not import web frameworks, database clients,
OpenHands SDK types, SAP SDKs, or cloud-provider SDKs. Infrastructure adapters
implement ports owned by the application layer.

## Coding Standards

### General design

- Prefer small, cohesive modules with one clear responsibility.
- Make dependencies explicit through constructors or function parameters.
  Avoid service locators, hidden globals, and import-time side effects.
- Use typed domain objects at boundaries. Do not pass unvalidated dictionaries
  through application code.
- Separate pure decision logic from I/O. Policy evaluation, configuration
  merging, state transitions, and assertion classification should be
  deterministic functions wherever possible.
- Represent important concepts explicitly: `TenantId`, `RunId`,
  `ManifestVersion`, `CapabilityId`, `OperationId`, `EvidenceRef`, and
  `ArtifactId` must not be interchangeable raw strings inside domain code.
- Prefer composition over inheritance and narrow interfaces over large manager
  classes.
- Keep functions readable and focused. Extract code when it clarifies a domain
  decision, enables isolated testing, or removes duplication; do not create
  abstractions without a concrete second use.
- Comments explain why, risk, or non-obvious constraints. Names and structure
  should explain what.
- Use UTC timestamps in storage and events, ISO 8601 at interfaces, and explicit
  timezone conversion only in presentation code.
- Use fixed-precision decimal types for costs and financial values. Never use
  binary floating point for money.
- Pin dependencies and runtime images. New dependencies require a clear need,
  license review, maintenance assessment, and security scanning.

### Naming

- Use product/domain language from the implementation plan consistently.
- Name capabilities as business actions, such as
  `read_code_metadata` or `execute_approved_test_step`, not by transport details.
- Name events in past tense, such as `run.created`, `manifest.resolved`, or
  `artifact.published`.
- Name commands as imperative actions and queries as read intent.
- Avoid ambiguous buckets such as `utils`, `helpers`, `common`, or `misc`.
  Place code with the domain or adapter that owns it.

### Errors and retries

- Use a documented error taxonomy: validation, authentication, authorization,
  policy, business, transient dependency, rate limit, timeout, conflict,
  uncertain external outcome, and terminal internal failure.
- Preserve safe diagnostic context and correlation identifiers without exposing
  credentials, tokens, personal data, confidential code, or raw model prompts.
- Retry only errors classified as transient and only at the layer that owns the
  operation.
- Use bounded exponential backoff with jitter.
- Never retry a write-like SAP or external operation until its prior outcome has
  been reconciled using the stable operation identifier.
- Do not catch broad exceptions merely to log and continue. Either handle the
  failure completely or propagate a typed error.

### Concurrency and state

- Assume messages and external events may be delivered more than once or out of
  order.
- Make event consumers idempotent and record deduplication keys durably.
- Use optimistic concurrency or atomic state transitions for run and approval
  updates.
- Do not keep authoritative run state only in process memory.
- State machines must reject illegal transitions and have table-driven tests.
- Cancellation must propagate to workflow activity, runtime conversation,
  in-flight tools where supported, and publication steps.

## Contracts and Configuration

### Runtime Manifest

Treat the Runtime Manifest as a first-class, versioned contract.

- Validate its schema before provisioning any resource.
- Resolve all inheritance before hashing and persistence.
- Canonicalize serialization so identical inputs produce identical hashes.
- Store the original request separately from the resolved manifest.
- Once execution starts, never update the manifest in place. A changed manifest
  creates a new run or an explicitly versioned continuation defined by policy.
- Resolve secret references only at the point of use; never materialize secret
  values in the manifest, logs, workspace, model context, or events.

### APIs and events

- Version external REST APIs, event envelopes, manifests, capability schemas,
  and persisted configuration formats.
- Validate input at the transport boundary and validate external output at the
  adapter boundary.
- Every request and event carries correlation, causation, run, and tenant
  context where applicable.
- Use pagination, limits, and stable ordering for collection endpoints.
- Maintain backward compatibility for active runs or provide an explicit,
  tested migration.
- Generate schemas and clients from one canonical contract when the selected
  toolchain supports it.

### Configuration

- Configuration may select only registered, approved implementations.
- Configuration must never contain executable scripts, arbitrary endpoints,
  unreviewed images, or controls that bypass policy, authorization, audit,
  redaction, or retention.
- Configuration changes are versioned and record author, approval status,
  effective date, compatibility, and content hash.
- Precedence is:
  `platform < use case < process/domain < tenant < SAP system < authorized request`.
- Reject unknown fields unless a compatibility rule explicitly allows them.
- Test resolution with golden fixtures covering overrides, conflicts, missing
  references, deprecation, and rollback.

## Data, Security, and Privacy

- MongoDB is the primary control-plane store for configuration and run metadata.
  All collections and indexes containing tenant data must include tenant scope.
- Published artifacts and evidence belong in versioned, encrypted,
  S3-compatible object storage with checksums and retention controls.
- Secrets belong in an enterprise secrets manager and should use short-lived
  credentials where supported.
- Audit data is append-only and stored independently of operational traces.
- Logs and telemetry are structured, correlated, and redacted by default.
- Never log authorization headers, cookies, secret references after resolution,
  credentials, full model prompts, raw confidential documents, or unfiltered SAP
  responses.
- Treat MCP output, retrieved documents, source comments, artifact content, and
  model output as untrusted input. Validate, constrain, and redact them.
- Enforce authorization on every resource lookup; possession of an identifier is
  never authorization.
- Production SAP writes are prohibited until separately approved through
  architecture and release governance.
- Do not use customer data for training or evaluation outside the tenant's
  explicitly approved policy.

## OpenHands and Workspace Rules

- Keep OpenHands behind a runtime adapter. Application and domain code must not
  depend on OpenHands event or persistence types.
- Associate every conversation with one platform run, tenant, user, workspace,
  and immutable manifest.
- Load only plugins, skills, MCP connections, model settings, and tools listed in
  the manifest.
- Normalize runtime events before exposing them to product clients or audit.
- Workspace images and plugins must be approved, versioned, scanned, and signed.
- Apply filesystem, network, CPU, memory, process, storage, and duration limits.
- The agent must checkpoint task progress, evidence indexes, decisions, and
  unresolved items outside active model context after each major task.
- Export durable artifacts before suspending, replacing, or destroying a
  workspace.
- Detect repeated actions, stalled progress, exhausted budgets, and circular
  searches.

## SAP Capability Rules

Each capability contract must define:

- Business-level name and purpose.
- Typed request and response schemas.
- Risk and data classification.
- Required tenant, role, environment, and scope permissions.
- Provider mapping and compatibility metadata.
- Timeout, retry, idempotency, and reconciliation behavior.
- Redaction and artifact-handling rules.
- Audit events and evidence produced.

Capability providers must be contract-tested. Test execution and any write-like
operation require stable operation IDs, non-production enforcement, cleanup
behavior, and deterministic reconciliation.

## Testing Strategy

Tests are part of the design, not a final cleanup step.

### Required test layers

- **Unit tests:** Domain invariants, policy decisions, state transitions,
  configuration merging, validation, redaction, and deterministic assertions.
- **Contract tests:** API, event, Runtime Manifest, MCP capability, provider, and
  artifact contracts.
- **Integration tests:** MongoDB, object storage, workflow engine, OpenHands,
  secrets manager, identity provider, and SAP provider adapters.
- **End-to-end tests:** Minimal request through manifest resolution, runtime
  execution, evidence validation, approval, publication, and cleanup.
- **Security tests:** Cross-tenant access, privilege escalation, prompt
  injection, malicious MCP output, secret leakage, artifact access, and network
  escape.
- **Reliability tests:** Duplicate and out-of-order events, worker/runtime death,
  credential expiry, partial responses, cancellation, restart, uncertain
  external outcomes, backup, and restore.
- **Evaluation tests:** Evidence coverage, unsupported claims, completeness,
  consistency, finding precision, and human-review acceptance.

### Test rules

- Every bug fix includes a regression test at the lowest effective layer.
- New behavior includes happy-path, authorization, validation, and failure-path
  tests.
- Do not mock the unit under test. Use fakes for owned ports and contract tests
  for external adapters.
- Tests must be deterministic: freeze time, seed randomness, and avoid live
  network dependencies unless explicitly marked as an integration environment.
- Tenant-sensitive features require at least one negative cross-tenant test.
- External effects require duplicate-delivery and uncertain-outcome tests.
- Schema changes require compatibility tests for active versions.
- Do not weaken or delete a test to make a change pass without documenting the
  changed requirement.

Before submitting a change, run the narrowest relevant checks and then the
repository's standard formatter, static analysis, unit tests, and affected
integration or contract tests. If a check cannot run, report exactly what was
not verified and why.

## Observability and Audit

Every user request, workflow, runtime conversation, capability call, SAP call,
and artifact must be traceable through stable correlation fields.

- Emit structured logs, traces, and metrics at service boundaries.
- Record latency, result class, retries, rate limits, resource usage, and cost
  without high-cardinality sensitive labels.
- Audit configuration activation, manifest creation, policy decisions,
  authorization decisions, tool calls, approvals, workflow transitions,
  artifact publication, and administrative actions.
- Audit records state who or what acted, tenant, time, target, configuration
  version, decision, and outcome.
- Operational telemetry may be sampled; required audit events may not.

## Documentation and Decisions

- Keep code, schemas, examples, and documentation in the same change.
- Add an ADR under `docs/adr/` for changes to component ownership, service
  boundaries, persistence technology, workflow semantics, external-effect
  guarantees, tenancy, authorization, public contracts, or deployment topology.
- ADRs contain context, decision, alternatives considered, consequences, status,
  and date.
- Update architecture diagrams when data flow, trust boundaries, or deployable
  components change.
- Document operationally significant behavior in a runbook, including failure
  detection, safe retry, reconciliation, recovery, and rollback.
- Do not state that a feature is secure, durable, idempotent, or production-ready
  without tests or operational evidence supporting the claim.

## Change Workflow for Coding Agents

For each task:

1. Read this file, the relevant implementation-plan sections, existing ADRs,
   nearby code, tests, and local instructions.
2. Restate the intended behavior and identify affected ownership boundaries.
3. Inspect the working tree and preserve unrelated user changes.
4. Choose the smallest coherent implementation that preserves the invariants.
5. Define or update contracts before implementing adapters.
6. Implement domain/application behavior before infrastructure wiring where
   practical.
7. Add tests for success, validation, authorization, failure, and recovery as
   applicable.
8. Run formatting, static analysis, tests, and relevant security or contract
   checks.
9. Review the diff for secrets, tenant leaks, accidental API changes,
   customer-specific logic, unsafe retries, and missing audit events.
10. Report what changed, what was verified, and any remaining risk or follow-up.

Do not make unrelated refactors, silently change architecture, introduce a new
framework, or broaden permissions as part of a feature fix. Do not commit,
push, deploy, migrate shared data, or change external systems unless the user
explicitly requests it.

## Definition of Done

A change is complete only when:

- The requested behavior and acceptance criteria are implemented.
- Architecture ownership and all non-negotiable invariants are preserved.
- Inputs, outputs, failures, authorization, tenant scope, and audit behavior are
  explicit.
- Tests cover the new behavior and relevant failure modes.
- Formatting, static analysis, and relevant tests pass.
- Public contracts and configuration remain compatible or have an approved,
  tested migration.
- Documentation and ADRs are updated where required.
- No secrets, customer-specific values, unrestricted endpoints, or unsafe
  capabilities were introduced.
- The final handoff identifies verification performed and any known limitation.

## Initial Delivery Priority

Until an ADR changes the sequence, prioritize:

1. Configuration catalogue and versioned Runtime Manifest contract.
2. Deterministic resolver, compatibility validation, and policy checks.
3. Run creation and durable workflow skeleton.
4. Runtime Manager and OpenHands conversation/workspace lifecycle.
5. Mock SAP capability through the governed MCP Gateway.
6. Audit, artifacts, progress streaming, cancellation, and cleanup.
7. One evidence-backed vertical slice through human approval and publication.
8. Failure, restart, isolation, idempotency, and recovery hardening.

Do not start multiple use-case implementations before the first vertical slice
meets its acceptance and recovery criteria.

---
> Source: [ash9119i9/TrueAI-harness](https://github.com/ash9119i9/TrueAI-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
