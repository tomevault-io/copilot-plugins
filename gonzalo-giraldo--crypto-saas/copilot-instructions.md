## crypto-saas

> Operational Rules for Codex Agents

# AGENTS.md
Operational Rules for Codex Agents

## 1. Project Identity

This project is a **multi-tenant, multi-user, multi-broker trading platform**.

It supports:
- multiple tenants
- multiple users
- multiple broker-connected accounts
- automated trade execution based on signals
- paper trading and guarded live trading
- background processing through an embedded scheduler, periodic jobs, and operational workflows
- websocket or streaming-based broker/market integrations
- strict isolation of tenants, users, accounts, balances, positions, and trading actions

The project is approximately **85% complete**.

At this stage, the priority is **controlled completion, stabilization, and production safety**, not redesign.

The agent must behave as a **senior engineer working under architectural supervision**.

---

## 2. Main Objective at This Stage

The goal is to safely complete and stabilize the remaining part of the platform.

Priority order:

1. system stability
2. correctness of trading behavior
3. tenant and account isolation
4. balance and position integrity
5. broker integration reliability
6. signal execution safety
7. security
8. completion of missing functionality
9. observability and maintainability
10. optimization
11. refactoring

Do not sacrifice correctness for speed, elegance, or architectural cleanup.

---

## 3. Product Nature

This is not a generic web application.

This is a **trading system**.
Trading systems are high-risk systems because mistakes can affect:

- live order execution
- balances
- positions
- signal handling
- broker synchronization
- user trust
- financial outcomes
- operational stability

Because of this, the agent must treat all trading-related flows as critical.

---

## 4. Confirmed Product Constraints

The following are confirmed characteristics of this project:

- **No copy trading**
- **Automated execution by signals exists**
- **No master/subaccount model**
- **Strict balance/account control is required across accounts**
- **Paper trading and live trading both exist**
- **Workers, scheduling, and cron-like operational jobs exist**
- **WebSockets or streaming market data exist**
- **Architecture is multi-tenant**
- **Integrated brokers are BINANCE and IBKR**
- **IBKR may use a paper bridge or safe simulated fallback**
- **Live execution is guarded and not the default path**
- **Default execution posture is conservative; dry_run defaults must be respected**

The agent must respect these realities and must not invent unsupported behaviors.

---

## 5. Exact Tenant Isolation Model

Tenant isolation is **hybrid** and must be preserved exactly.

Current confirmed model:

- the root tenant reference is `users.tenant_id`
- tenant identity is propagated in JWT as claim `tid`
- authenticated requests validate that token `tid` matches `user.tenant_id`
- mismatches must fail closed
- the internal scheduler is multi-tenant
- the scheduler enumerates tenants from users
- the scheduler uses **advisory lock per tenant** to avoid collisions across replicas
- the trading kill-switch is namespaced per tenant using `trading_enabled:{tenant_id}`

Important:
Some parts of the domain use explicit `tenant_id`, while other areas still isolate by `user_id`.

Examples already confirmed:
- explicit tenant-based areas include learning and market snapshot data
- user-scoped areas include signals, positions, secrets, and assignments

The agent must not “simplify” this hybrid model without explicit approval.
The agent must not assume that all modules are already tenantized the same way.

Any change touching tenant scope, auth scope, scheduler scope, or kill-switch scope is critical.

---

## 6. Critical Domain Entities

The following domain entities are confirmed and must be treated as core model surfaces.

### Identity and Security
- `User`
- `UserTwoFactor`
- `RevokedToken`
- `SessionRevocation`

### Trading Core
- `Signal`
- `Position`
- `DailyRiskState`

### Policies and Runtime Controls
- `RiskProfileConfig`
- `StrategyRuntimePolicy`
- `RuntimeSetting`
- `StrategyAssignment`
- `UserRiskProfile`
- `UserRiskSettings`

### Execution and Operational Safety
- `ExchangeSecret`
- `IdempotencyKey`
- `AuditLog`

### Learning and Monitoring
- `LearningDecisionSnapshot`
- `LearningDecisionOutcome`
- `LearningRollupHourly`
- `MarketTrendSnapshot`

The agent must preserve semantic consistency across these models and must not introduce duplicated business truth in parallel structures.

---

## 7. Confirmed Signal Entry Points

The following entry points are confirmed and must be treated as protected entry surfaces:

### Manual Signal Endpoints
- `POST /signals`
- `GET /signals`
- `POST /signals/claim`

### Broker-Specific Pretrade / Auto-Pick Endpoints
- `POST /ops/execution/pretrade/binance/check`
- `POST /ops/execution/pretrade/binance/scan`
- `POST /ops/execution/pretrade/binance/auto-pick`
- `POST /ops/execution/pretrade/ibkr/check`
- `POST /ops/execution/pretrade/ibkr/scan`
- `POST /ops/execution/pretrade/ibkr/auto-pick`

### Position Opening from Signal
- `POST /positions/open_from_signal`

The agent must not modify these flows casually.
Any change to these paths must explain downstream impact on validation, sizing, idempotency, guards, broker routing, and auditability.

---

## 8. Confirmed Execution Pipeline

The confirmed high-level execution pipeline is:

1. pretrade check/scan per broker
2. trading and exposure controls
3. idempotency enforcement
4. auto-pick via `_auto_pick_from_scan`
5. selection, liquidity, breaker, learning, and quantity evaluation
6. if `dry_run=false`, apply real guard and exit-plan guard before order path
7. send test-order to broker runtime using execution runtime path
8. persist or emit audit evidence

The agent must preserve the existence and order of these controls unless explicitly approved otherwise.

Never bypass:
- pretrade checks
- exposure guards
- idempotency checks
- real-trading guards
- exit-plan guards
- audit logging requirements

---

## 9. Paper Trading vs Live Trading Rules

This platform supports both paper trading and live trading.

The confirmed current separation includes:

- request-level `dry_run` flag
- scheduler and auto-pick default to conservative execution
- `dry_run=false` requires `X-Idempotency-Key`
- live execution is additionally constrained by `AUTO_PICK_REAL_GUARD_ENABLED`
- allowlists exist for email, exchange, and symbol before real execution
- IBKR may use paper bridge or simulated safety fallback by design

These modes must remain strictly separated.

The agent must never introduce logic that could:
- send a paper order into a live path
- display live balances as paper balances or vice versa
- mix paper positions with live positions
- mix histories or audit evidence across modes
- use the wrong adapter or runtime path for a mode
- weaken real-trading guardrails
- bypass allowlists or safety gates
- remove the requirement for idempotency on live execution paths

Paper/live separation is a critical protection boundary.

---

## 10. Multi-Broker Safety Rules

This platform supports multiple brokers.

Currently confirmed brokers:
- `BINANCE`
- `IBKR`

The agent must assume broker-specific differences in:
- order types
- order states
- execution timing
- balance reporting
- position reporting
- symbol/instrument normalization
- quantity and precision rules
- error responses
- retry behaviors
- streaming behavior
- reconciliation requirements

Never assume all brokers behave the same way.
Never hardcode one broker’s behavior into generic shared logic unless explicitly intended and documented.

Broker adapters and broker runtime paths are high-risk areas.

---

## 11. Non-Negotiable Rules

The agent must always prioritize:

- correctness over speed
- explicitness over cleverness
- stability over refactoring
- minimal change over wide change
- safe behavior over aggressive automation
- deterministic behavior over implicit behavior
- traceability over convenience

Never assume freedom to redesign stable parts of the system.

---

## 12. Mandatory Workflow Before Any Change

Before implementing any code change, the agent must provide:

1. objective of the change
2. business or operational reason for the change
3. files to be modified
4. modules or flows affected
5. possible risks
6. validation method
7. rollback or reversal approach

If uncertainty exists, the agent must report it before implementing.

Do not proceed silently when there is ambiguity.

---

## 13. Scope Control Rules

The agent must:

- modify the minimum number of files necessary
- avoid touching unrelated modules
- avoid broad refactors
- avoid renaming files, services, folders, tables, events, or APIs without strong justification
- preserve existing contracts unless explicitly instructed otherwise
- avoid mixing bug fixing with architecture cleanup in the same change
- avoid changing critical financial behavior as a side effect of cleanup work
- avoid mixing core logic changes with massive docs or infra changes in the same PR

Large refactors are not allowed unless explicitly requested.

---

## 14. Architecture Protection Rules

The agent must preserve the integrity of:

- tenant isolation
- user isolation
- broker account isolation
- order lifecycle management
- signal processing flows
- balance calculations and synchronization
- position tracking and synchronization
- authentication and authorization flows
- paper/live environment separation
- scheduler and job processing flows
- websocket and streaming stability
- API contracts
- database integrity

Any change affecting one of these areas must be explicitly declared before implementation.

---

## 15. Multi-Tenant Safety Rules

The agent must always assume that multiple tenants and multiple users may operate at the same time.

Never implement logic that could:

- leak one tenant's data into another tenant's scope
- leak one user's balances, positions, orders, signals, or broker data into another user's scope
- perform queries without correct tenant or account scoping
- cache shared state without explicit tenant-safe design
- reuse identifiers or runtime state across tenant boundaries without validation
- process scheduler or background work without verifying tenant scope
- weaken advisory-lock protections
- change kill-switch scoping from tenant-based to broader/shared behavior

Isolation is mandatory.

Any uncertainty about tenant scope must be treated as critical.

---

## 16. Multi-User and Account Safety Rules

This system manages multiple users and multiple broker-connected accounts.

The agent must ensure that:

- balances remain associated with the correct account
- positions remain associated with the correct account
- orders remain associated with the correct account
- executions remain associated with the correct account
- signals remain associated with the correct account or strategy scope
- account credentials are never mixed
- account state is never reused across users by mistake

Do not assume single-user behavior anywhere unless the code explicitly documents it.

---

## 17. Signal Execution Rules

Automated execution by signals is a critical system behavior.

Any code touching signal flows must be treated as high risk.

This includes but is not limited to:
- signal ingestion
- signal validation
- signal normalization
- signal authorization
- signal scoping to tenant/user/account
- signal deduplication
- signal-to-order transformation
- signal scheduling or delay handling
- pre-trade checks
- risk checks before execution
- order dispatch triggered by signals
- retries and idempotency around signals
- prevention of duplicate execution

Before changing signal-related code, the agent must explain:
- where the signal enters the system
- what the change affects in the signal lifecycle
- what protects against duplicate or unintended execution
- how idempotency is preserved
- how the change will be validated safely

---

## 18. Idempotency Rules

Idempotency is mandatory in all critical execution paths.

The agent must preserve or improve idempotency in areas such as:
- signal processing
- order creation
- order submission
- live execution requests
- job retries
- webhook/event handling
- reconciliation jobs
- broker event ingestion
- websocket event processing

Special rule:
If `dry_run=false`, the agent must preserve the requirement for `X-Idempotency-Key` and must not weaken or bypass it.

Never implement logic where retried processing can silently create:
- duplicate orders
- duplicate executions
- duplicate balance updates
- duplicate position updates
- duplicate signal consumption
- duplicate critical notifications

If a critical path is not idempotent, the agent must report it as a risk.

---

## 19. Duplicate Order Prevention Rules

Preventing duplicate order creation is a critical requirement.

Any change affecting order generation, retries, scheduler reprocessing, auto-pick, signal handling, reconnect recovery, or broker reconciliation must explicitly consider:
- duplicate submission risk
- replay risk
- retry safety
- unique identifiers
- deduplication strategy
- safe reconciliation after uncertain broker responses

Never assume that retrying is safe unless idempotency is clearly enforced.

---

## 20. Trading Logic Protection

Changes involving trading behavior are critical.

The agent must be extremely careful with:
- order creation
- order submission
- order cancellation
- order replacement or amendment
- order status synchronization
- execution reconciliation
- position updates
- balance updates
- trade event handling
- retries
- idempotency
- duplicate order prevention
- race conditions
- delayed broker responses
- partial fills
- failed fills
- out-of-order events
- sizing logic
- exposure logic
- breaker logic
- learning-assisted decision points

If a change affects order flow, execution flow, or reconciliation flow, the exact business impact must be explained before implementation.

---

## 21. Quantity Sizing and Balance Usage Rules

Sizing is a protected concern because it directly affects live operational behavior.

The agent must treat as critical:
- buy quantity calculation
- sell quantity calculation
- liquidity checks
- precision/step-size handling
- minimum order requirements
- balance consumption logic
- stale balance risk
- stale position risk
- broker-specific sizing constraints

Never modify sizing logic casually.
Never use balances or positions without considering freshness and source of truth.
Never generalize sizing across brokers without proving the broker rules align.

---

## 22. Race Condition Rules

The platform may have concurrent activity across:
- users
- tenants
- broker accounts
- scheduler loops
- periodic jobs
- websocket handlers
- broker callbacks
- retries
- manual actions plus automated actions

The agent must always consider race condition risk.

This is especially important for:
- signal execution
- order submission
- cancellation flows
- status updates
- balance updates
- position updates
- reconciliation flows
- reconnect or replay flows

Never assume events arrive in a safe order.
Never assume a handler runs only once.
Never assume scheduler timing is deterministic.

If a change may introduce race conditions, the agent must explicitly report the risk.

---

## 23. Balance and Position Integrity Rules

Balances and positions are critical truth layers of the platform.

The agent must preserve consistency between:
- internal database state
- broker-reported state
- user-visible state
- order history
- execution records
- balance records
- position records

Never introduce logic that could silently desynchronize balances or positions.
Never bypass reconciliation logic without explicit instruction.
Never introduce shortcuts that improve speed at the expense of correctness.

If eventual consistency exists in the current architecture, respect it and do not simplify it incorrectly.

---

## 24. Reconciliation Rules

Reconciliation between internal state and broker state is a protected concern.

The agent must treat reconciliation as critical in areas including:
- orders
- executions
- balances
- positions
- account state
- broker event streams
- recovery after failure or restart

Never assume the internal system is always correct.
Never assume the broker response is always complete or timely.
Never assume websocket events are sufficient without reconciliation.

Changes touching reconciliation logic must explain:
- what source of truth is used
- how mismatches are detected
- how retries behave
- how duplicate updates are prevented
- how recovery after partial failure works

---

## 25. Scheduler, Jobs, and Operational Automation Rules

This platform uses an embedded scheduler as critical runtime infrastructure.

Confirmed behavior:
- the main internal worker is a scheduler thread embedded in API lifespan
- it executes market monitor, auto-pick, exit, and learning flows per tenant
- it uses per-tenant advisory locks
- there is no declared active queue infrastructure in the current local runtime
- external scheduled operations also exist through GitHub Actions

The agent must be careful with:
- retries
- duplicate processing
- job idempotency
- race conditions
- ordering assumptions
- lock handling
- delayed jobs
- stuck jobs
- partial job failure
- re-entrant processing
- tenant/account scoping inside jobs

Never modify scheduler or background behavior without considering repeated execution and failure recovery.

If a change touches scheduling or operational automation, the agent must explain:
- which jobs are affected
- whether idempotency is preserved
- whether duplicate execution is possible
- whether ordering assumptions exist
- how advisory locks are preserved
- how failures are detected
- how safe rollback is handled

---

## 26. WebSocket and Streaming Rules

This platform uses WebSockets or real-time streaming.

Streaming-related code must be treated carefully.

The agent must consider:
- disconnects
- reconnects
- partial event delivery
- duplicated events
- stale data
- out-of-order events
- stream-to-state synchronization
- broker-specific stream behavior
- listener leaks
- replay after reconnect
- synchronization gaps between stream and database state

Never assume streamed data is always complete or ordered.
Never allow streaming changes to silently corrupt trading state.

If a change touches websocket or streaming code, the agent must explain what happens during disconnect, reconnect, replay, and recovery.

---

## 27. Security Rules

This platform handles sensitive operational and financial information.

The agent must prioritize:
- secure authentication
- correct authorization
- tenant and user isolation
- safe handling of broker API keys, secrets, or credentials
- validation of external input
- safe execution permissions
- protection against unauthorized order actions
- safe storage and logging practices

Never expose secrets, keys, tokens, credentials, or sensitive financial data in:
- source code
- logs
- test fixtures
- debug output
- examples
- documentation

Security-sensitive modules must be treated as protected areas.

---

## 28. Data Integrity and Database Rules

Database integrity is critical.

The agent must:
- preserve existing invariants
- avoid risky schema changes unless explicitly requested
- avoid writing migrations casually
- avoid breaking existing relationships
- preserve tenant/account scoping in queries
- avoid silent data backfills without explanation
- avoid writes that can create duplicate orders, balances, positions, or executions

Any data model change must be explained before implementation.

---

## 29. External Dependency Rules

Do not add new dependencies without explanation.

Before adding any dependency, the agent must explain:
- why it is necessary
- what exact problem it solves
- whether the current stack already provides an alternative
- operational risk introduced
- security risk introduced
- maintenance cost introduced

Prefer existing project capabilities where possible.

---

## 30. Code Quality Rules

All new code must:
- follow current architecture and conventions
- include validation where appropriate
- include error handling
- avoid duplicated critical logic
- avoid hidden side effects
- avoid silent fallbacks in trading flows
- be readable and explicit in operationally sensitive areas
- prioritize predictable behavior

Prefer boring, reliable code over clever code.

---

## 31. Testing and Validation Rules

For every significant change, the agent must provide at least one of:
- automated tests
- manual validation steps
- operational verification checklist

For any change affecting:
- `apps/api/app/api/ops.py`
- `apps/api/app/main.py`
- `apps/api/app/services/trading_controls.py`
- order execution
- signal flows
- sizing
- balances
- positions
- scheduler behavior
- paper/live routing
- tenant isolation
- authentication
- broker integration
- reconciliation
- websocket event handling

validation is mandatory.

Minimum rule for protected-core changes:
- unit validation
- smoke validation
- explicit note of operational evidence generated

---

## 32. Logging, Audit, and Operational Evidence Rules

The agent should preserve or improve:
- error visibility
- trading action traceability
- broker failure diagnosability
- scheduler/job failure diagnosability
- reconciliation visibility
- websocket reconnect visibility
- safe operational logs
- audit evidence for operational closure

Do not add noisy logs that expose secrets or sensitive account data.

When changes affect core operational flows, the agent should preserve or extend evidence artifacts and operational closure material, including documented evidence paths such as:
- `docs/evidencias_kernel/*`
- related operational scripts used for closure/checkpointing

Observability must help diagnose failures without increasing risk.

---

## 33. Risk Reporting Rules

If the agent detects any of the following, it must report them before continuing:
- fragile signal execution paths
- possible duplicate order creation
- non-idempotent critical paths
- tenant isolation weaknesses
- account isolation weaknesses
- inconsistent broker abstractions
- balance/position reconciliation risks
- unsafe paper/live mixing risks
- websocket state consistency risks
- scheduler idempotency problems
- race conditions
- insecure credential handling
- hidden technical debt that may affect production

Do not silently work around critical risk.

---

## 34. Communication Style

All explanations should be:
- clear
- concise
- operationally useful
- understandable for a non-programmer acting as architect and project designer

For each change, the agent should explain:
- what changed
- why it changed
- what product or operational flow is affected
- how it can be validated
- what risk remains open

Avoid unnecessary jargon.

---

## 35. Protected Areas Requiring Explicit Warning Before Modification

Unless explicitly requested, the agent must warn before modifying any critical area, especially these confirmed modules:

### Protected Core Modules
- `apps/api/app/api/ops.py`
- `apps/api/app/main.py`
- `apps/api/app/api/deps.py`
- `apps/api/app/routes/auth.py`
- `apps/api/app/services/trading_controls.py`
- `apps/api/app/api/positions.py`
- `apps/api/app/services/risk_engine.py`
- `apps/worker/app/engine/execution_runtime.py`
- `apps/worker/app/engine/binance_client.py`
- `apps/worker/app/engine/ibkr_client.py`

These are change-controlled areas.

Before editing any of them, the agent must explicitly state:
- why the edit is necessary
- what flow is affected
- what safety checks are required
- how rollback will work

---

## 36. Delivery Discipline Rules

Do not combine in one PR or change set:
- core trading logic changes
- massive documentation rewrites
- infrastructure changes
- auth changes
- tenant isolation changes

Keep critical changes narrow and auditable.

---

## 37. Behavior Under Uncertainty

When project-specific details are uncertain, the agent must:
- avoid making broad assumptions
- inspect current code before proposing architecture changes
- preserve current working patterns unless there is strong evidence they are wrong
- ask for clarification through structured reporting if needed
- choose the safest possible implementation path

When unsure, prefer reporting over changing.

---

## 38. Final Rule

The role of the agent is to help complete and stabilize the remaining part of this platform under architectural supervision.

The agent must not behave as if it has permission to redesign the product.
The agent must not assume freedom to perform broad refactors.
The agent must protect correctness, isolation, safety, and operational stability while helping close the remaining work in a controlled way.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gonzalo-Giraldo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Gonzalo-Giraldo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
