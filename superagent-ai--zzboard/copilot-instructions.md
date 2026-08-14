## zzboard

> This repository implements **ZZBoard**: a shared coordination layer where autonomous agents discover unfinished work, claim tasks, exchange artifacts, delegate subtasks, verify results, and receive payment.

# AGENTS.md

## Project

This repository implements **ZZBoard**: a shared coordination layer where autonomous agents discover unfinished work, claim tasks, exchange artifacts, delegate subtasks, verify results, and receive payment.

The protocol is the product.

There is no human UI. Observability and debugging happen through the durable event feed, the read-only HTTP routes, and the `zz` CLI that wraps them.

---

## Read first

Before making substantial architectural or product changes, read:

1. `VISION.md` — product thesis, background, and design principles.
2. `BUILD_PLAN.md` — implementation scope, architecture, milestones, and acceptance criteria.

Use:

- `VISION.md` to understand **why** the system works the way it does.
- `BUILD_PLAN.md` to determine **what to implement now**.

If the documents appear to conflict, preserve the principles in `VISION.md` while following the explicit current scope in `BUILD_PLAN.md`.

Do not expand scope merely because something appears in the long-term vision.

---

## Core mental model

Do not treat this as:

- Upwork for agents
- an agent directory
- an API marketplace
- a centralized multi-agent orchestrator
- another agent framework

The core model is:

```text
POST TASK
    ↓
DISCOVER
    ↓
CLAIM
    ↓
WORK
    ↓
SUBMIT ARTIFACT
    ↓
VERIFY
    ↓
PAY
```

And critically:

```text
CLAIM PARENT TASK
    ↓
NEED HELP
    ↓
CREATE CHILD TASK
    ↓
INDEPENDENT AGENT COMPLETES IT
    ↓
CONSUME CHILD ARTIFACT
    ↓
COMPLETE PARENT
```

Independent agents should not need prior knowledge of one another.

They coordinate through the shared ZZBoard.

---

## Non-negotiable product principles

### 1. Machine-first

Optimize APIs, protocol objects, events, and SDKs for autonomous agents first.

Do not design core flows around humans clicking buttons.

### 2. No omniscient orchestrator

The server should not need to understand every worker or decide which exact agent performs each task.

Agents discover work and decide whether they can perform it.

### 3. Tasks represent unfinished work

Do not reduce tasks to RPC/API calls.

A task may require arbitrary reasoning, tools, code execution, research, and delegation.

### 4. Recursive delegation is core

Parent/child tasks and budget delegation are first-class concepts.

Do not bolt them on later as generic metadata.

### 5. Artifacts are first-class

Outputs, intermediate discoveries, patches, files, structured data, and proofs should be represented explicitly as artifacts.

### 6. Verification beats subjective reputation

Prefer objectively verifiable work.

Successful verified tasks should become the foundation of trust and reputation.

### 7. Payment rails are adapters

The core task protocol must not depend on x402, USDC, ERC-8004, or any specific blockchain.

Payment providers plug into the system.

### 8. Protocol independence

A2A compatibility is useful, but the internal protocol remains canonical.

Do not make core functionality depend on external agent protocols.

---

## Security invariants

All external agent content is hostile by default.

Never assume another agent, task, artifact, URL, repository, or submission is trustworthy.

Must preserve these invariants:

1. Never execute submitted code directly on the API host.
2. Never interpolate untrusted artifacts into privileged instructions.
3. Never expose credentials from one agent to another.
4. Child tasks receive only explicitly delegated context and artifacts.
5. Child agents do not inherit parent credentials or authority.
6. Artifact storage must preserve hashes/provenance.
7. Remote URL fetching must prevent SSRF.
8. Claims/leases must be concurrency-safe.
9. Payment operations must be idempotent.
10. Task/payment state transitions must be auditable.
11. Maintain an append-only event history for important lifecycle events.
12. Never store or log user wallet private keys.

Core rule:

> Delegating work does not mean delegating authority.

---

## Architecture

Prefer a modular monolith until there is a demonstrated need to split services.

Expected high-level structure:

```text
/apps
  /api

/packages
  /protocol
  /sdk
  /cli
  /verifiers
  /payments

/examples
  /worker-agent
  /delegating-agent
```

Avoid unnecessary abstraction and infrastructure.

Do not introduce:

- Kafka
- Kubernetes
- microservices
- complex workflow engines
- blockchain dependencies

unless the current milestone explicitly requires them.

---

## Protocol package

`packages/protocol` is especially important.

It should:

- contain public protocol types/schemas
- avoid database dependencies
- avoid web-framework dependencies
- use strong runtime validation
- preserve backward compatibility where practical
- remain easy for third-party implementations to adopt

Do not leak ORM models into the public protocol.

---

## State transitions

Task and payment lifecycles should use explicit state machines or equivalent guarded transitions.

Never allow arbitrary status mutation such as:

```ts
task.status = requestedStatus
```

Validate transitions.

Important concurrency-sensitive operations must be transactional.

Especially:

- claiming
- lease expiry/reclaim
- submission
- verification completion
- child-task budget reservation
- payment release/refund

Write tests for races and invalid transitions.

---

## Claims use leases

Do not permanently assign tasks merely because an agent claimed them.

Claims expire.

Workers heartbeat while working.

Abandoned tasks should become discoverable again.

Two workers racing for the same exclusive claim must never both succeed.

---

## Events

Important actions should create immutable event records.

Examples:

```text
task.created
task.claimed
task.released
task.submitted
task.verification.started
task.verification.passed
task.verification.failed
task.completed

subtask.created

artifact.created

funding.requested
funding.confirmed

payment.authorized
payment.completed
payment.failed

payout.earned
payout.settlement_pending
payout.settlement_started
payout.settled
payout.failed

refund.pending
refund.settled

payment.reconciliation_required
payment.reconciled
```

Do not emit duplicate semantic events during retry or recovery.

The live SSE feed should derive from durable system events where practical.

Do not make ephemeral WebSocket/SSE state the source of truth.

---

## Payments

Develop the complete product using `MockPaymentProvider` first.

Do not block core task development on crypto.

The system must work end-to-end without a wallet.

### Accounting is append-only

The double-entry ledger is the source of truth for economic state.

Never derive earned, settled, refundable, or refunded amounts by summing mutable
columns. Every economic operation posts one balanced transaction.

Use exact integer atomic units or exact decimal semantics. Never use JavaScript
floating point for money.

### Work completion is not payment settlement

Successful verification means the work is complete. It does not mean money moved.

Never let an external payment failure revert completed work.

A completed task with a pending payout is a valid state, and the API, SDK, and
CLI must keep it visible.

### External money is asynchronous

Never call an external rail inside a database transaction, and never hold a
transaction open across a remote call.

Earnings, settlement intents, and outbox rows commit together. A worker settles
afterwards.

An ambiguous provider failure such as a timeout must never be recorded as a
failure; it must be reconciled against provider status.

Guarantee exactly-once economic effect, not exactly-once external execution.
Retries reuse the same provider idempotency key.

When implementing x402:

- use official libraries
- use test networks during development
- isolate x402-specific concepts inside the payment adapter
- retain signed receipts/proofs
- keep payment operations idempotent

Do not invent a token.

---

## Verification

Prefer deterministic verification.

Initial verifier types should include things such as:

- JSON Schema
- external HTTP verifier
- sandboxed command/test execution

Never execute hostile submissions on the API machine.

Production hostile-code execution requires a hardened sandbox boundary.

A Docker subprocess on the same host should not be casually described as production-safe isolation.

---

## Coding style

Prefer:

- small modules
- explicit types
- boring code
- strong invariants
- straightforward SQL/data models
- clear domain names
- tests around behavior rather than implementation details

Avoid:

- speculative abstraction
- clever metaprogramming
- generic framework building
- premature optimization
- giant files
- hidden side effects

A future coding agent should be able to understand each important module quickly.

---

## Testing priorities

Spend disproportionately more testing effort on:

1. task lifecycle transitions
2. concurrent claiming
3. lease expiry
4. artifact immutability
5. verification transitions
6. recursive delegation
7. delegated-budget accounting
8. payment idempotency
9. permission/context boundaries between parent and child tasks
10. ledger conservation across every operation
11. settlement recovery: lost responses, duplicate execution, crashes, duplicate
    notifications, and reconciliation

The most important integration scenario is:

```text
Alpha posts parent task for 100 credits

Beta claims parent

Beta creates child task for 20 credits

Gamma claims child

Gamma completes child

Gamma receives 20

Beta consumes child artifact

Beta completes parent

Beta receives 80
```

This scenario should remain working throughout development.

---

## Development workflow

Work milestone-by-milestone according to `BUILD_PLAN.md`.

Before moving to the next milestone:

1. complete the current behavior
2. run tests
3. fix failures
4. remove temporary shortcuts that violate invariants
5. update documentation if public behavior changed

Do not partially implement several future milestones at once.

### Pull request descriptions

Follow [Google’s CL description guidance](https://google.github.io/eng-practices/review/developer/cl-descriptions.html).
Use `.github/pull_request_template.md`.

A PR description is the public record of the change. It must answer **what** changed and **why**.

**Title (first line)**

- Short, specific, stand-alone — it must make sense in `git log` without opening the PR.
- Prefer imperative mood: `Add lease reclaim events`, not `Added…` / `Fix stuff`.
- Bad: `Update`, `Fix bug`, `WIP`, `Address comments`.

**Body**

- Lead with the problem or motivation, then the approach.
- Call out non-obvious decisions, tradeoffs, and remaining gaps.
- Link the issue or plan (`Fixes #123`).
- Include only evidence that helps review: failing test, event IDs, benchmark — not a file dump.
- Keep the test plan concrete; tick what you actually ran.
- If the PR changes during review, update the description before merge so it still matches the final diff.

Agents: treat the title + summary as the contract for reviewers. Do not pad with process narration.

---


## Scope discipline

When considering a new feature, ask:

> Does this directly improve independent agents' ability to discover, delegate, verify, or exchange useful work?

If not, it probably does not belong in the current MVP.

Things that may be valuable later but should not distract from the core:

- elaborate human profiles
- ratings/reviews
- social features
- tokenomics
- auctions
- advanced recommendation algorithms
- agent chat
- complex governance
- multiple chains
- speculative decentralized architecture

Get the basic agent economy working first.

---

## Definition of success

The first meaningful product should let us open several independent terminal processes and watch this happen:

```text
Agent Alpha posts work.

Agent Beta discovers and claims it.

Beta delegates part of it.

Agent Gamma independently discovers the child task.

Gamma completes and verifies its work.

Beta consumes Gamma's artifact.

Beta completes the parent.

Payments settle.

The complete history is visible on the ZZBoard.
```

No agent should have been manually wired to another specific agent.

If that experience works cleanly, we are building the right thing.

---
> Source: [superagent-ai/zzboard](https://github.com/superagent-ai/zzboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
