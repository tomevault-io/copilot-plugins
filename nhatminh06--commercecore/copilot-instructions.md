## commercecore

> handles 10k users

# CLAUDE.md

## Project

CommerceCore is a backend-only e-commerce system for studying correctness under concurrency and failure.

The focus is not storefront development.

The focus is:

```text
inventory
transactions
checkout
payments
idempotency
events
failure
recovery
```

Long-term progression:

```text
Catalog + Inventory
        ↓
Cart
        ↓
Inventory Reservations
        ↓
Orders + Checkout
        ↓
Idempotency
        ↓
Payments
        ↓
Transactional Outbox
        ↓
Event Delivery
        ↓
Saga / Compensation
        ↓
Reconciliation
        ↓
Failure Experiments
```

## Stack

Primary stack:

```text
Java 21
Spring Boot
Gradle
PostgreSQL
JUnit
Testcontainers
```

Local dependencies may use:

```text
Docker
Docker Compose
```

Kafka and Redis may be introduced later only when an implemented requirement justifies them.

Kubernetes is not part of the core project.

## Architecture

Begin as a modular monolith.

One application.

One PostgreSQL database.

Keep domain boundaries clear.

Do not split modules into microservices merely because they might eventually communicate over a network.

Distributed boundaries should be introduced deliberately so their costs can be studied.

## Primary priorities

Optimize for:

1. business correctness
2. transactional correctness
3. concurrency correctness
4. idempotency
5. explicit state transitions
6. reproducible failures
7. integration tests using real dependencies
8. understandable code

Do not optimize for architecture complexity.

## Git policy

Do not commit unless explicitly instructed.

Do not push unless explicitly instructed.

Never:

```text
force push
rewrite history
rebase without approval
stage unrelated files
```

Before a requested commit report:

```text
files changed
diff summary
tests
proposed commit message
```

## No AI attribution

Never add references to:

```text
Claude
Anthropic
ChatGPT
OpenAI
Copilot
AI-generated
generated-by
assisted-by
```

Never add an AI system as an author, contributor, or co-author.

Never add AI `Co-Authored-By` trailers.

## Avoid AI slop

Do not automatically create:

```text
Controller
Service
Repository
DTO
Mapper
Interface
Impl
Factory
Manager
```

for every domain concept.

Every layer must have a reason.

Avoid:

```text
BaseCrudService
GenericRepository
CommerceManager
BusinessEngine
DomainManager
```

and other abstractions that hide the actual commerce behavior.

Prefer explicit domain concepts.

## Spring

Use Spring as infrastructure.

Do not allow Spring conventions to obscure business invariants.

The question:

> Why is this operation correct?

should be answerable from the actual domain/database behavior, not simply:

> because Spring handles it.

## PostgreSQL

PostgreSQL is the authoritative persistent store initially.

Use real PostgreSQL for persistence/concurrency integration tests.

Do not replace it with H2 when PostgreSQL semantics matter.

Use Flyway for schema migrations.

Do not depend on Hibernate auto-creating the production schema.

## Database invariants

Protect important invariants at the database layer when appropriate.

Examples:

```text
unique SKU
inventory >= 0
unique idempotency key
unique external payment event
```

Application checks alone may race.

Database constraints are part of the design.

## Concurrency

Do not solve cross-request correctness with only:

```java
synchronized
```

or another process-local lock.

Correctness should survive multiple application instances whenever the database can enforce it naturally.

Use explicit PostgreSQL concurrency mechanisms.

Examples may include:

```text
conditional UPDATE
row locking
unique constraint
optimistic versioning
```

Choose based on the actual invariant.

Do not introduce distributed locks automatically.

## Transactions

A transaction must protect a specific invariant.

Do not add `@Transactional` everywhere by habit.

For each important transaction be able to explain:

```text
what must change atomically?
what failure would occur without the transaction?
```

## Money

Never use:

```text
double
float
```

for money.

Use a deliberate representation such as:

```text
BigDecimal with explicit scale
```

or integer minor units.

Document currency assumptions.

Do not build multi-currency support before needed.

## Inventory

Inventory must never become negative.

Cart quantity is not inventory reservation.

A cart does not guarantee stock.

Later reservation state must be explicit.

Do not conflate:

```text
cart
available inventory
reserved inventory
sold inventory
```

## Idempotency

Retries are normal backend behavior.

When checkout idempotency exists, correctness must persist across application restart.

Do not implement idempotency using only process-local memory.

Distinguish:

```text
same request retried
```

from:

```text
new logical operation
```

## Payments

Treat payment providers as external systems that may:

```text
timeout
retry
duplicate callbacks
respond late
return ambiguous outcomes
```

Never assume a timeout means payment failed.

Do not blindly retry potentially successful payment operations.

Later use reconciliation when provider outcome is ambiguous.

## Webhooks

Assume webhook delivery is at least once.

Handlers must tolerate duplicates.

Do not describe idempotent processing as network-level exactly-once delivery.

## Outbox

When event publication becomes necessary, explicitly address the dual-write problem.

Preferred pattern:

```text
domain state
+
outbox event

same database transaction
```

Then publish asynchronously.

Do not publish an event before its corresponding database state commits.

## Events

Domain events should describe committed domain facts.

Prefer:

```text
OrderCreated
PaymentAuthorized
InventoryReserved
```

over vague events such as:

```text
OrderUpdated
DataChanged
```

when specificity improves semantics.

Do not introduce Kafka before events have a concrete role.

## Kafka

Kafka is optional until event-delivery requirements exist.

Do not use Kafka as résumé decoration.

Once added, test:

```text
duplicates
retries
consumer restart
ordering assumptions
poison messages
```

Do not assume Kafka solves business idempotency automatically.

## Redis

Redis is optional.

Introduce it only when a concrete requirement exists.

Possible later uses:

```text
cache
rate limiting
short-lived auxiliary state
```

Do not move correctness-critical source-of-truth state into Redis casually.

## Microservices

Do not start with microservices.

If a service is extracted later, document:

```text
what was previously a local call?
what is now a network call?
what new failure modes appeared?
```

Service extraction is an experiment in distributed boundaries, not a project-size metric.

## Frontend

No frontend is required.

Do not add:

```text
React
Vue
Angular
storefront
CSS work
```

unless explicitly requested later.

Use:

```text
HTTP API
curl
HTTPie
integration tests
```

for interaction.

## Kubernetes

Do not add Kubernetes during core development.

Do not create:

```text
Helm
Deployments
Services
Ingress
Kustomize
```

unless deployment itself becomes a later explicit requirement.

CommerceCore is a backend correctness project, not a Kubernetes project.

## Docker

Docker is useful for reproducible dependencies.

Docker Compose may run local infrastructure.

Testcontainers should provide isolated integration dependencies during tests.

Do not require developers to manually start Docker Compose before every integration test when Testcontainers can provide the dependency.

## Testing

Important behaviors must have integration tests where the database participates.

Particularly:

```text
inventory concurrency
idempotency
transaction rollback
outbox atomicity
duplicate webhooks
workflow compensation
```

Do not mock away the component whose semantics are being tested.

## Concurrency tests

Use synchronization primitives to coordinate concurrent test operations.

Prefer:

```text
CountDownLatch
CyclicBarrier
ExecutorService
```

where appropriate.

Do not rely on random sleeps to create races.

Test invariants, not which thread wins.

## Failure experiments

Failure is part of CommerceCore.

Eventually deliberately test:

```text
concurrent inventory exhaustion
duplicate checkout
duplicate payment event
worker crash
reservation expiration
payment timeout after charge
application restart
```

Each experiment should state the invariant it verifies.

## Business errors

Expected business rejection is not necessarily a server failure.

Examples:

```text
insufficient stock
expired reservation
invalid state transition
duplicate logical request
```

Model these deliberately.

Do not turn all expected outcomes into HTTP 500.

## State machines

Important domain objects should have explicit legal transitions.

Examples later include:

```text
Order
Payment
InventoryReservation
```

Do not allow arbitrary state assignment if transition semantics matter.

Do not build a generic state-machine framework.

## Documentation

README should answer:

```text
what CommerceCore is
what currently works
what invariant is currently demonstrated
how to run it
how to test it
what is intentionally unsupported
```

Do not advertise future milestones as completed.

Do not use marketing claims such as:

```text
enterprise-grade
production-ready
highly scalable
fault-tolerant
```

without evidence.

## Observability

Do not add Prometheus, Grafana, or OpenTelemetry simply because backend systems often use them.

Introduce observability later if a concrete failure experiment or measurement needs it.

## Security

Do not begin with authentication.

Authentication is not the difficult correctness problem CommerceCore is currently studying.

Add it only if a later feature genuinely requires user identity/authorization semantics.

## Scope discipline

Follow the active milestone.

Do not create future modules in advance.

If an unrelated issue is discovered:

1. explain it
2. identify whether it blocks current work
3. propose the smallest correction
4. do not silently refactor unrelated areas

## No fake scale claims

Do not report:

```text
handles 10k users
scales horizontally
high throughput
```

without a reproducible load test.

Concurrency correctness experiments are not throughput benchmarks.

## Learning requirement

This project is being built so the user understands backend engineering, not merely so code exists.

After every meaningful milestone, provide a concise learning walkthrough.

Include:

```text
What changed?
Why is it needed?
Which invariant does it protect?
How do the pieces interact?
```

## Code walkthrough

Identify only the 2–5 most useful code locations to read.

For each explain:

```text
what it does
why it matters
what backend concept to notice
```

Do not ask the user to read the whole repository.

## Commands to run

After each milestone provide exact copyable commands.

Potential examples:

```bash
./gradlew test
docker compose up -d
./gradlew bootRun
curl ...
```

Commands must match actual implementation.

Never invent endpoints or flags.

## Database inspection

When a milestone involves persistence, provide useful SQL commands so the user can inspect the actual state.

Examples:

```sql
SELECT * FROM inventory;
```

must use actual table names.

Help the user connect application behavior to stored state.

## Check understanding

After each milestone give 3–5 short questions with short answers.

Focus on important backend concepts.

Examples:

```text
Why is synchronized insufficient for stock correctness?

Why is a database unique constraint stronger than an application pre-check?

Why can a payment timeout be ambiguous?

What problem does the transactional outbox solve?
```

## Hands-on experiment

After each meaningful milestone give one 5–15 minute experiment using the code that now exists.

Do not perform the entire experiment for the user.

Let them interact with the system.

## Before implementation

For non-trivial work briefly state:

1. current behavior
2. intended change
3. invariant being protected

Then implement.

Do not produce long speculative planning.

## After implementation

Report:

### Changed

Actual components changed.

### Invariant

What correctness property is now enforced?

### Design

What mechanism enforces it?

### Verification

Exact commands/tests and outcomes.

### Limitations

What is still intentionally unsupported?

### Learning

Provide the required walkthrough.

## Definition of done

A backend feature is not complete merely because an endpoint responds.

It should normally have:

```text
defined invariant
database constraints where appropriate
transaction semantics
business failure behavior
integration tests
concurrency/failure test if relevant
accurate documentation
narrow diff
no unsupported claims
learning walkthrough
commands to reproduce
```

## Project mindset

When choosing between:

```text
more endpoints
```

and:

```text
proving one transaction is correct
```

prove correctness.

When choosing between:

```text
more microservices
```

and:

```text
understanding one failure boundary
```

understand the boundary.

When choosing between:

```text
more infrastructure
```

and:

```text
better checkout semantics
```

choose checkout semantics.

When choosing between:

```text
architecture that looks impressive
```

and:

```text
behavior that survives concurrency and failure
```

choose behavior.

---
> Source: [nhatminh06/commercecore](https://github.com/nhatminh06/commercecore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
