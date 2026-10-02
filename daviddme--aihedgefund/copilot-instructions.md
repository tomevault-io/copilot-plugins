## aihedgefund

> generateStructured<TInput, TOutput>(

# CLAUDE.md

This file is the operating guide for Claude Code and any coding agent working in this repository.

Read this file before changing code.

---

## 1. Project mission

Build **ARF-OS**, a multi-agent research operating system for discovering, implementing, backtesting, validating, and forward-testing systematic trading strategies.

The system is a research platform. It is not an autonomous fund, broker, or live-trading engine.

The core product promise is:

> Every strategy result is reproducible, versioned, independently validated, and traceable from idea to decision.

Pine Script v6 is the canonical strategy language. A Pine-compatible local runner may be used for scale, but TradingView is the final acceptance and parity environment for Pine behaviour.

---

## 2. Read these documents

Before implementing a feature, read:

1. `AI_RESEARCH_HEDGE_FUND_SPEC.md`
2. `LEADER_AGENT_SYSTEM_PROMPT.md`
3. Relevant schemas in `schemas/`
4. Relevant ADRs in `docs/adr/`
5. Existing tests near the code you will change

When implementation and specification conflict, do not silently choose one. Open or update an ADR and make the conflict visible.

---

## 3. Non-negotiable architecture rules

### 3.1 Immutable research artefacts

Never mutate a tested strategy version.

Any change to any of the following creates a new `strategy_versions` row:

- Pine source
- Strategy definition
- Parameters
- Symbol
- Timeframe
- Session or timezone
- Cost model
- Position sizing
- Leverage or margin
- Execution settings
- Dataset
- Runner
- Segment assignment

Backtest and forward-test records point to the exact immutable version.

### 3.2 API owns state transitions

Workers do not directly change strategy lifecycle state.

Workers:

1. execute a job,
2. store output artefacts,
3. emit a domain event or result,
4. let the orchestrator/API apply transition policy.

### 3.3 Structured contracts are canonical

All agent outputs, handoffs, domain events, and external signal payloads must be validated with Zod.

Do not accept critical free-form model output and then infer fields with regular expressions.

Prose summaries may accompany structured data, but structured data is canonical.

### 3.4 Separation of duties

Enforce role separation in code:

- Creator cannot be sole validator.
- Validator cannot edit source.
- Strategy Judge cannot grant live approval.
- Forward operator cannot change active strategy parameters.
- Human override must be explicit and audited.

### 3.5 Protected data

Final holdout and forward data require explicit access checks.

Never return protected results through a general strategy endpoint without verifying role and stage.

Every protected-data read writes an audit event.

### 3.6 Idempotency

Every side-effecting command and background job must be idempotent.

Use:

- request idempotency keys,
- deterministic job keys,
- unique database constraints,
- event IDs,
- deployment signal IDs.

Retries must not create duplicate strategy versions, backtests, paper orders, fills, or decisions.

### 3.7 No hidden business logic in prompts

Prompts may guide agent behaviour, but promotion gates, access control, lifecycle transitions, budgets, and hard-fail rules live in deterministic application code.

A model recommendation is evidence, not authority.

### 3.8 No browser automation as a core dependency

The MVP supports human-assisted TradingView verification and CSV ingestion.

Do not build the platform around fragile UI selectors or unattended browser automation unless an approved ADR explicitly documents terms, reliability, and security implications.

### 3.9 No live trading in the initial product

Do not add exchange credentials, order-routing code, or live execution side effects without a separately approved project specification.

Paper execution must be clearly named and isolated.

---

## 4. Technology choices

Use the current stable versions pinned in the repository lockfile and runtime configuration.

### Applications

- `apps/web`: Next.js App Router
- `apps/api`: Fastify
- `apps/worker-research`: model and research jobs
- `apps/worker-backtest`: runner and report ingestion jobs
- `apps/worker-analytics`: metrics and robustness jobs
- `apps/worker-forward`: TradingView webhook and paper-test jobs

### Packages

- TypeScript
- pnpm workspaces
- Turborepo
- PostgreSQL
- Drizzle ORM
- Redis
- BullMQ
- Zod
- Clerk
- OpenTelemetry
- Vitest
- Playwright
- S3-compatible object storage

Avoid introducing a large framework when a small internal abstraction is sufficient.

The agent orchestration state machine belongs to ARF-OS. Do not hide it inside a third-party agent framework.

---

## 5. Expected repository layout

```text
/
├── apps/
│   ├── web/
│   ├── api/
│   ├── worker-research/
│   ├── worker-backtest/
│   ├── worker-analytics/
│   └── worker-forward/
├── packages/
│   ├── contracts/
│   ├── db/
│   ├── agent-runtime/
│   ├── workflow/
│   ├── metrics/
│   ├── pine/
│   ├── backtest-sdk/
│   ├── event-bus/
│   ├── auth/
│   ├── observability/
│   └── ui/
├── pine/
│   ├── boilerplate/
│   ├── libraries/
│   ├── fixtures/
│   └── generated/
├── schemas/
├── docs/
│   └── adr/
├── infra/
├── AI_RESEARCH_HEDGE_FUND_SPEC.md
├── LEADER_AGENT_SYSTEM_PROMPT.md
└── CLAUDE.md
```

Do not create arbitrary top-level folders.

---

## 6. Local commands

Use package scripts rather than invoking tools with ad hoc flags.

Expected commands:

```bash
pnpm install
pnpm dev
pnpm build
pnpm lint
pnpm typecheck
pnpm test
pnpm test:integration
pnpm test:e2e
pnpm db:generate
pnpm db:migrate
pnpm db:seed
pnpm format
```

Before marking work complete, run at minimum:

```bash
pnpm lint
pnpm typecheck
pnpm test
```

For API, database, queue, or workflow changes, also run integration tests.

For user-visible flows, run relevant Playwright tests.

If a command does not exist yet, add it consistently at the root and affected workspace.

---

## 7. TypeScript standards

### 7.1 Strictness

Use strict TypeScript.

Do not use:

- `any`
- unchecked type assertions
- non-null assertions unless proved by a local invariant
- untyped JSON
- stringly typed lifecycle states
- floating promises
- swallowed errors

Prefer:

- discriminated unions
- branded IDs
- exhaustive switches
- `unknown` plus validation
- pure functions for calculations
- dependency injection at service boundaries

### 7.2 IDs

Use UUIDv7-compatible IDs for new domain records.

Represent domain IDs as branded strings in TypeScript where practical:

```ts
type StrategyVersionId = string & { readonly __brand: "StrategyVersionId" };
```

Do not pass raw IDs across unrelated aggregate functions without typed boundaries.

### 7.3 Dates and timezones

- Store timestamps in UTC.
- Use ISO 8601 at API boundaries.
- Store market-session timezone separately.
- Never rely on the server’s local timezone.
- Do not use JavaScript `Date` arithmetic for market windows without a timezone-aware library or tested helper.
- Include explicit date-boundary tests around daylight saving transitions.

### 7.4 Money and percentages

- Do not use binary floating point for authoritative monetary totals.
- Use database numeric types and a decimal library.
- Store percentages with clear semantics.
- Name fields to distinguish `0.05` from `5`.
- Include units in contracts where ambiguity exists.

### 7.5 Errors

Use typed domain errors.

API errors follow a problem-details shape with:

- `type`
- `title`
- `status`
- `detail`
- `instance`
- `code`
- `traceId`
- optional validation errors

Never expose secrets, model prompts, SQL, stack traces, or provider credentials to clients.

---

## 8. Contract design

All contracts live in `packages/contracts`.

Every contract requires:

- Zod schema
- inferred TypeScript type
- version
- examples
- tests
- backward-compatibility notes when changed

Contracts include:

- Agent task
- Agent output
- Handoff
- Strategy Definition
- Parameter Manifest
- Backtest Plan
- Backtest Run result
- Metric Snapshot
- Validation Report
- Forward Signal
- Paper Order and Fill
- Committee Decision
- Domain Event

Use explicit enum values. Do not silently coerce invalid values.

---

## 9. Database rules

### 9.1 Source of truth

PostgreSQL is authoritative for:

- identities,
- workflow state,
- strategy registry,
- metadata,
- decisions,
- audit,
- job references.

Object storage is authoritative for large immutable artefacts such as:

- Pine source bundles,
- CSV exports,
- raw reports,
- logs,
- compressed equity series,
- chart exports.

### 9.2 Migrations

- Every schema change uses a migration.
- Migrations are forward-safe.
- Do not edit an applied migration.
- Destructive migrations require an ADR and staged rollout.
- Add indexes for real query patterns, not speculation.
- Use database constraints for invariants where possible.

### 9.3 Transactions

Use a transaction for:

- State transition plus audit event
- Strategy-version creation plus lineage
- Decision plus status change
- Signal acceptance plus idempotency record
- Paper fill plus equity update

Use an outbox table when a domain event must be emitted reliably after a transaction.

### 9.4 Audit

Audit tables are append-only through the application.

Every audit record includes:

- actor,
- action,
- aggregate,
- prior state summary,
- new state summary,
- reason,
- timestamp,
- trace ID.

---

## 10. Workflow engine rules

The state machine lives in `packages/workflow`.

It must provide:

- Allowed transitions
- Required evidence by transition
- Required role
- Human approval requirement
- Hard-fail checks
- Policy version
- Idempotent transition command
- Decision record generation

Never scatter transition checks across route handlers.

Example:

```ts
const result = await workflow.transition({
  strategyVersionId,
  from: "ROBUSTNESS_VALIDATION",
  to: "TRADINGVIEW_VERIFICATION",
  actor,
  evidenceIds,
  policyVersion,
});
```

The workflow returns a typed success or failure result. It does not throw for expected policy rejection.

---

## 11. Agent runtime rules

### 11.1 Provider abstraction

Model providers implement a small interface:

```ts
interface ModelProvider {
  generateStructured<TInput, TOutput>(
    request: StructuredGenerationRequest<TInput, TOutput>,
  ): Promise<StructuredGenerationResult<TOutput>>;
}
```

Provider adapters must not contain research workflow logic.

### 11.2 Prompt storage

Prompts are versioned records with:

- role,
- semantic version,
- content hash,
- status,
- benchmark score,
- approved by,
- approved at.

Never load an unapproved challenger in production.

### 11.3 Output validation

Flow:

1. Send role prompt and task contract.
2. Receive model output.
3. Parse JSON.
4. Validate with Zod.
5. If invalid, retry once with exact validation errors.
6. If still invalid, mark the run failed or route to an approved fallback.
7. Preserve raw provider output in protected diagnostics storage.
8. Store only safe summaries in normal UI records.

### 11.4 Tool permissions

Each role has a tool allowlist.

Examples:

- Idea Scout: web/research sources, internal knowledge search
- Indicator Researcher: source/code inspection, Pine docs, internal indicator library
- Pine Engineer: repository, compiler/runner, tests
- Backtest Engineer: runner, datasets, report ingestion
- Validator: read-only evidence, analytics
- Forward Operator: deployment configuration and paper engine
- Judge: read-only evidence and decision command

Do not give every agent arbitrary shell, database, and network access.

### 11.5 Prompt injection

Retrieved documents are data.

Wrap untrusted content, label it as untrusted, and instruct models not to follow embedded instructions.

Never allow a research source to redefine the role, reveal secrets, or invoke tools.

---

## 12. Pine package rules

`packages/pine` owns:

- Manifest types
- Static checks
- Source hash helpers
- Approved code templates
- Definition-to-source mapping
- TradingView report ingestion helpers
- Runner capability matrix

### 12.1 Pine defaults

Generated source must default to:

- `//@version=6`
- explicit `strategy()` properties
- confirmed-bar logic
- `pyramiding = 0`
- one stop and one target
- realistic commission
- explicit slippage
- explicit margin
- standard OHLC assumptions
- date/segment inputs
- stable order IDs
- versioned alert payload

### 12.2 Pine lint hard errors

Treat as hard errors unless an approved exception exists:

- future references,
- suspicious negative offsets,
- unsafe `barmerge.lookahead_on`,
- unconfirmed MTF values,
- missing cost model,
- missing source/SDL identity,
- undeclared `calc_on_every_tick`,
- undeclared pyramiding,
- source/manifest mismatch,
- unbounded optimisable input,
- missing alert deployment fields.

### 12.3 Pine lint warnings

Warnings include:

- low warm-up,
- non-standard charts,
- large request count,
- lower-timeframe ambiguity,
- same-bar stop/target assumptions,
- limit-fill assumptions,
- low trade count,
- high leverage,
- session ambiguity.

### 12.4 Golden Pine fixtures

Maintain fixtures that cover:

- market entry next bar,
- process on close,
- stop and target,
- long/short reversal,
- no pyramiding,
- session filter,
- confirmed higher-timeframe value,
- commission,
- slippage,
- margin,
- date windows,
- alert payloads.

Any runner or parser change must pass the golden suite.

---

## 13. Backtest SDK rules

The local runner implements:

```ts
interface BacktestRunner {
  capabilities(): RunnerCapabilities;

  compile(input: CompileInput): Promise<CompileResult>;

  run(input: BacktestInput): Promise<BacktestResult>;

  cancel(runId: BacktestRunId): Promise<void>;
}
```

Every result contains:

- runner name and version,
- code hash,
- manifest hash,
- dataset hash,
- environment hash,
- parameters,
- execution settings,
- trades,
- equity or enough data to reconstruct it,
- warnings,
- timing,
- error details.

Do not normalise away runner-specific warnings. Preserve them.

---

## 14. Metric calculation rules

`packages/metrics` provides independent calculations.

Requirements:

- Pure deterministic functions
- Explicit units
- Calculation version
- Tests against hand-calculated fixtures
- Tests against TradingView exports where applicable
- No silent dropping of NaN, missing trades, or zero-duration periods

Metrics must state scope:

- run,
- segment,
- strategy version,
- symbol,
- parameter set,
- forward deployment,
- portfolio.

Do not compare metrics across incompatible scopes without a clearly named aggregation.

---

## 15. TradingView report ingestion

### 15.1 Uploads

Use presigned object-store uploads.

Validate:

- allowed file type,
- size,
- virus/malware policy where applicable,
- expected verification task,
- checksum,
- uploader identity.

### 15.2 Parsing

TradingView exports may vary by tab and locale.

The parser must:

- identify report type,
- detect delimiter and locale safely,
- map columns through versioned adapters,
- reject ambiguous numeric formats,
- preserve raw upload,
- emit parser warnings,
- never guess an unknown column’s meaning.

### 15.3 Parity

Parity comparison starts with identity:

1. Pine source hash
2. Manifest/settings
3. Symbol
4. Timeframe
5. Date range
6. Costs
7. Sizing
8. Execution mode
9. Trade sequence

Report the first divergence, not only aggregate metric differences.

---

## 16. Forward-test rules

### 16.1 Signal endpoint

The TradingView webhook endpoint must:

- use a deployment-specific high-entropy token,
- validate JSON,
- validate deployment state,
- validate strategy version,
- validate symbol/timeframe,
- check timestamp tolerance,
- calculate idempotency key,
- store the raw safe payload,
- enqueue processing,
- return quickly.

Do not execute paper logic synchronously in the webhook request.

### 16.2 Paper engine

Paper execution is deterministic and versioned.

A deployment records:

- fill-model version,
- latency model,
- slippage model,
- fees,
- quantity model,
- stop and target rules.

Do not edit a running deployment’s model. Create a new deployment.

### 16.3 Health

A deployment may be:

- active and healthy,
- active and degraded,
- paused,
- failed,
- completed.

Infrastructure degradation must be separate from strategy performance.

---

## 17. API rules

### 17.1 Routes

Keep route handlers thin:

1. authenticate,
2. authorise,
3. validate,
4. call application service,
5. map typed result,
6. return.

No SQL or workflow rules in route handlers.

### 17.2 Pagination

Use cursor pagination for large collections.

### 17.3 Filtering

Validate filters. Use indexed fields. Do not expose arbitrary SQL-like filtering.

### 17.4 SSE

Use SSE for:

- campaign updates,
- job progress,
- forward deployment health.

Events must be resumable with event IDs where practical.

### 17.5 Idempotency

Commands accept an `Idempotency-Key` header.

Persist the key, actor, request hash, and response reference.

Reject key reuse with a different request body.

---

## 18. Frontend rules

### 18.1 Evidence clarity

Always label:

- in-sample,
- validation,
- final holdout,
- forward,
- gross,
- net,
- simulated,
- paper,
- TradingView,
- local runner.

Never present historical and forward equity as one uninterrupted series without a visible boundary.

### 18.2 Immutable views

Tested Pine revisions are read-only.

Editing creates a child version.

### 18.3 Human decisions

Decision dialogs must show:

- exact strategy version,
- mandatory evidence status,
- validator recommendation,
- hard failures,
- strongest rejection case,
- override status.

Do not allow a one-click approval that hides the evidence.

### 18.4 Charts

Use a dedicated time-series library for equity and drawdown and a capable chart library for heatmaps and distributions.

Requirements:

- linked date brushing,
- accessible summaries,
- export,
- no misleading dual axes,
- scope and units in tooltip,
- empty/error/stale states.

### 18.5 Data fetching

- Server components for stable read views where useful.
- Client fetching for live operations.
- Central typed API client.
- No direct database access from the web app.
- No duplicated contract types.

---

## 19. Security rules

- Never log secrets.
- Never send exchange or infrastructure secrets to a model.
- Use least privilege for service accounts.
- Protect object paths by organisation.
- Verify organisation ownership on every aggregate access.
- Rotate webhook tokens.
- Rate limit public endpoints.
- Apply CSRF protection where relevant.
- Use secure cookies.
- Validate redirect URLs.
- Sanitize rendered markdown.
- Do not render arbitrary HTML from model output.
- Preserve a tamper-evident audit trail.

Security-sensitive changes require focused tests and review.

---

## 20. Observability rules

Use structured logs.

Every background operation includes:

- trace ID,
- correlation ID,
- job ID,
- campaign ID,
- strategy version ID,
- actor or service,
- duration,
- outcome,
- error code.

Instrument:

- API latency,
- queue depth,
- job latency,
- model cost,
- model schema failure,
- backtest duration,
- parity failure,
- webhook error,
- signal duplicate,
- stale deployment.

Do not rely on logs alone for user-visible workflow status. Persist status in the database.

---

## 21. Testing expectations

### 21.1 Unit

Add unit tests for:

- schemas,
- policies,
- transitions,
- metrics,
- parsing,
- hashing,
- idempotency,
- Pine lints,
- segment generation.

### 21.2 Integration

Use real PostgreSQL and Redis in integration tests where behaviour matters.

Test:

- outbox delivery,
- transaction rollback,
- unique constraints,
- queue retry,
- object ingestion,
- API auth and ownership,
- protected-data audit.

### 21.3 End-to-end

For user-visible changes, add or update a Playwright path.

### 21.4 Regression

Every bug fix requires a test that fails before the fix and passes after it.

---

## 22. Implementation workflow for coding agents

For every task:

1. Restate the concrete goal internally.
2. Locate the specification section and existing code.
3. Inspect relevant tests and contracts.
4. Identify invariants and affected aggregates.
5. Make the smallest coherent change.
6. Add or update tests.
7. Run focused tests.
8. Run lint, typecheck, and relevant suite.
9. Review the diff for:
   - accidental mutation,
   - missing audit,
   - missing authorisation,
   - missing idempotency,
   - unvalidated JSON,
   - unhandled states,
   - data leakage.
10. Summarise:
   - files changed,
   - behaviour changed,
   - tests run,
   - remaining risks.

Do not refactor unrelated code unless required for correctness.

---

## 23. Git rules

- Use small, intentional commits.
- Do not mix generated files with unrelated source changes.
- Do not commit secrets, local databases, report exports, or model raw outputs.
- Update documentation when behaviour or contracts change.
- Use conventional commit prefixes if the repository adopts them.
- Never force-push a shared branch.
- Do not bypass failing tests to merge.

---

## 24. ADR rules

Create an ADR when deciding:

- workflow technology,
- queue technology,
- analytical datastore,
- runner implementation,
- TradingView automation,
- protected-data model,
- prompt-evaluation method,
- metric formula change,
- live-execution scope,
- major security boundary.

ADR structure:

```text
Title
Status
Context
Decision
Alternatives
Consequences
Security implications
Migration/rollback
```

---

## 25. Definition of done

A feature is done only when:

- contracts exist,
- implementation exists,
- authorisation exists,
- audit exists where required,
- idempotency exists where required,
- errors are typed,
- tests pass,
- docs are updated,
- UI has loading/empty/error states,
- observability exists,
- no secrets or protected data leak,
- the feature matches the specification.

For a strategy-research feature, also confirm that it preserves immutable lineage and protected-data rules.

---

## 26. Never do these things

- Never mutate a tested strategy version.
- Never let a worker bypass the workflow engine.
- Never infer required fields from invalid model prose.
- Never hide a failed backtest.
- Never tune on final holdout.
- Never mark reused data as unseen.
- Never merge local-runner and TradingView results without a parity report.
- Never make screenshot data canonical when CSV or structured data exists.
- Never let an agent approve its own work.
- Never expose chain-of-thought in the UI.
- Never put live exchange credentials in this project’s initial scope.
- Never add browser automation without an ADR and terms review.
- Never use `any` to silence a design problem.
- Never disable tests to make a change pass.
- Never add a model framework that owns critical state implicitly.
- Never represent a backtest as a guarantee.

---

## 27. First implementation sequence

Unless the repository already contains later phases, build in this order:

1. Contracts and IDs
2. Database and migrations
3. Auth and organisation boundaries
4. Audit log
5. Strategy and version registry
6. Campaign and task workflow
7. Agent-run storage and structured handoffs
8. Pine source and manifest storage
9. TradingView CSV upload and parser
10. Independent trades/equity/metrics
11. Strategy Library and Strategy Detail
12. Backtest Lab
13. Validation policy and reports
14. TradingView parity
15. Forward webhook and paper engine
16. Practice arena
17. Portfolio research

Do not start with a visually impressive dashboard backed by fake or mutable data.

---

## 28. Final coding-agent instruction

Optimise for correctness, auditability, and reproducibility.

The platform should make it easier to reject a weak strategy than to make it look strong.

When a requested implementation would weaken lineage, data protection, role separation, or reproducibility, stop and surface the conflict instead of coding around it.

---
> Source: [daviddme/AIHedgeFund](https://github.com/daviddme/AIHedgeFund) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
