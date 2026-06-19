## ddd-foundations

> >


# DDD Foundations Skill

The layering map and term definitions for any DDD-Quarkus bounded context. Every other DDD skill in this set
references the four layers and the canonical names below.

**Foundational principle.** Domain depends on nothing in your code — only the JDK and your own domain types.
Infrastructure and interfaces depend inward. The word "service" without a qualifier is meaningless: say
*application service* (use-case orchestration, owns the transaction, returns DTOs) or *domain service*
(stateless domain logic that doesn't naturally belong to one aggregate), and put each in the layer named after
it. The compiler enforces these layers via package boundaries — violations show up as imports, not as code
review findings.

**Red Flags — STOP if you find yourself thinking:**

- "Repository = persistence = infrastructure, so the interface goes there too."
- "It feels tidy to put the interface next to its implementation."
- "It's just a service — `service/` is fine for both kinds."
- "We're a small team / it's a small BC, let's keep it flat in one package and split later."
- About to write `import jakarta.persistence.*` or `import jakarta.ws.rs.*` in a class under `<bc>.domain`.
- About to write `import com.example.<bc>.infrastructure.*` from a class under `<bc>.domain` or `<bc>.application`.
- About to name a new class `FooService` without `Application` or `Domain` in the name.

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing the package layout for a new bounded context.
- Placing a class (entity, value object, repository, domain service, application service, JAX-RS resource,
  ExceptionMapper).
- Naming a "service" class — choosing the layer-qualified suffix.
- Reviewing a diff that imports `jakarta.persistence.*` or `jakarta.ws.rs.*` in a `<bc>.domain` class.
- Reviewing a diff that puts a Repository interface in `<bc>.infrastructure`.
- Reviewing a diff where a JAX-RS resource injects a repository directly (skipping the application service).

**Out of scope**: the Quarkus mechanics inside any one layer (use the corresponding `quarkus-*` skill);
aggregate-internal design — invariants, factories, identity strategy (use `ddd-aggregates`); domain-event payload
and dispatch rules (use `ddd-domain-events`); choosing between active-record and repository pattern at the
persistence layer (architectural decision; the persistence skill takes the *Quarkus mechanics* of either path).

---

## Core Rules

1. **Four layers per bounded context**: `domain`, `application`, `infrastructure`, `interfaces` (often with a
   `interfaces.rest` sub-package). One bounded context = one top-level package; one set of layers under it.
2. **Domain depends on nothing in your code.** Only JDK types and types under `<bc>.domain` itself.
3. **Application depends on domain.** Application services orchestrate use cases by calling domain types and
   domain interfaces.
4. **Infrastructure depends on domain.** Infrastructure provides *implementations* of domain interfaces
   (repositories, external clients).
5. **Interfaces depend on application.** REST resources / gRPC handlers / CLI commands call application services
   and pass DTOs back and forth.
6. **Never the reverse.** Domain must not import application, infrastructure, or interfaces. Application must not
   import infrastructure or interfaces.
7. **Repository interfaces live in `<bc>.domain`**, alongside the aggregate they serve. The interface mentions
   only domain types — no `jakarta.persistence`, no Panache, no Hibernate.
8. **Repository implementations live in `<bc>.infrastructure`.** They implement the domain interface and bring in
   framework concerns. CDI wires them at runtime.
9. **Application service** = use-case orchestrator. Owns the transaction boundary (`@Transactional`). Returns
   DTOs, never entities. Lives in `<bc>.application`. Class name ends in `ApplicationService` (e.g.
   `AuthorizePaymentApplicationService`).
10. **Domain service** = stateless domain logic that doesn't naturally belong to one aggregate. No I/O, no
    framework calls, takes domain types and returns domain types. Lives in `<bc>.domain`. Class name ends in
    `DomainService` (e.g. `FraudDetectionDomainService`).
11. **Never name a class just `FooService`.** The unqualified name is the layer-confusion smell that lets
    transaction logic and domain rules drift between files in a shared `service/` folder.
12. **JAX-RS resources live in `<bc>.interfaces.rest`** and depend only on `<bc>.application`. They never
    `@Inject` a repository or an entity directly.
13. **DTOs (commands and read models) live in `<bc>.application`.** They cross the application↔interfaces
    boundary. The domain stays in domain types.

---

## Canonical Example

The package layout for a `payments` bounded context. Use case: *authorize a payment* — load the `Payment`
aggregate, run a domain-service fraud check, advance the aggregate's state machine, persist via a Panache
implementation, return a `PaymentDTO`.

```
com.example.payments
├── domain
│   ├── PaymentMethod              (value object — Java record)
│   ├── Payment                    (aggregate root)
│   ├── PaymentRepository          (interface — uses only domain types)
│   ├── FraudDetectionDomainService
│   └── PaymentAuthorized          (domain event — record)
├── application
│   ├── AuthorizePaymentCommand    (input DTO)
│   ├── PaymentDTO                 (output DTO)
│   └── AuthorizePaymentApplicationService
├── infrastructure
│   └── PanachePaymentRepository   (implements domain.PaymentRepository)
└── interfaces
    └── rest
        ├── PaymentsResource
        └── PaymentNotFoundMapper  (ExceptionMapper)
```

Allowed dependency directions (one way only):

```
interfaces.rest ──▶ application ──▶ domain
                                       ▲
                            infrastructure
```

Sketch of the four key files:

```java
// com.example.payments.domain.PaymentRepository
package com.example.payments.domain;

import java.util.Optional;
import java.util.UUID;

public interface PaymentRepository {
    Optional<Payment> findById(UUID id);
    Optional<Payment> findByOrderId(OrderId orderId);
    void persist(Payment p);
}
```

```java
// com.example.payments.infrastructure.PanachePaymentRepository
package com.example.payments.infrastructure;

import com.example.payments.domain.Payment;
import com.example.payments.domain.PaymentRepository;
import io.quarkus.hibernate.orm.panache.PanacheRepositoryBase;
import jakarta.enterprise.context.ApplicationScoped;
import java.util.UUID;

@ApplicationScoped
public class PanachePaymentRepository
        implements PaymentRepository, PanacheRepositoryBase<Payment, UUID> {
    // ... Panache helpers; CDI wires this as the PaymentRepository implementation
}
```

```java
// com.example.payments.application.AuthorizePaymentApplicationService
package com.example.payments.application;

import com.example.payments.domain.*;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;

@ApplicationScoped
public class AuthorizePaymentApplicationService {

    @Inject PaymentRepository payments;            // domain interface, not the impl
    @Inject FraudDetectionDomainService fraud;     // domain service

    @Transactional
    public PaymentDTO authorize(AuthorizePaymentCommand cmd) {
        Payment p = payments.findById(cmd.paymentId()).orElseThrow(PaymentNotFoundException::new);
        FraudVerdict verdict = fraud.check(p, cmd.method());
        p.authorize(verdict);
        payments.persist(p);
        return PaymentDTO.of(p);
    }
}
```

```java
// com.example.payments.interfaces.rest.PaymentsResource
package com.example.payments.interfaces.rest;

import com.example.payments.application.AuthorizePaymentApplicationService;
// ... imports from application; never from domain or infrastructure directly
```

What this demonstrates:

- The repository **interface** lives in `domain`. Its method signatures use only domain types (`Payment`,
  `OrderId`, `UUID`, `Optional`). No Panache, no Hibernate, no Jakarta Persistence imports.
- The Panache **implementation** lives in `infrastructure`. It implements the domain interface and brings the
  framework concerns. `@ApplicationScoped` plus CDI lets the application service depend on the domain interface,
  not the implementation.
- The application service in `application/` orchestrates: load → call domain service → mutate aggregate →
  persist → map to DTO. It owns the `@Transactional` boundary (per `quarkus-persistence`).
- The fraud-detection logic lives in `domain` because it is pure domain logic with no framework dependencies and
  doesn't naturally belong inside one aggregate. Suffix `DomainService` makes that explicit.
- The resource lives in `interfaces.rest` and depends only on `application`. It never sees `Payment`,
  `PaymentRepository`, or `PanachePaymentRepository`.

---

## Anti-patterns

| Don't | Why it's wrong | Fix |
|---|---|---|
| Put `PaymentRepository` (interface) in `<bc>.infrastructure` next to `PanachePaymentRepository` | Inverts the dependency: `domain.Payment` would have to import from infrastructure to refer to its own persistence contract. The layering map breaks. | Move the interface to `<bc>.domain` next to the aggregate it serves. Only the implementation belongs in `infrastructure`. |
| Name a class `OrderService` (no qualifier) | Hides whether it's orchestration or domain logic. The two end up in the same folder and rules drift between them — transaction handling creeps into rule logic, or rules creep into orchestration. | Pick the qualifier and the layer: `OrderApplicationService` in `application/` *or* `OrderDomainService` in `domain/`. |
| Domain class importing `jakarta.persistence.*` or `jakarta.ws.rs.*` | Domain now depends on the persistence framework / HTTP framework. ORM upgrades, mapping-strategy changes, and column-naming policies cascade through the model. The "domain depends on nothing" rule is broken at the source. | Domain stays a plain POJO. The JPA-annotated mirror class lives in `<bc>.infrastructure` and the repository implementation translates between domain aggregates / value objects and persistence types (covered in `ddd-aggregates`, `ddd-value-objects`, and `ddd-repositories`). |
| Application service importing `<bc>.infrastructure` types directly | Inverts dependency direction; `application → infrastructure` is forbidden. The application now knows about Panache. | Application imports the *domain interface* (`PaymentRepository`); CDI wires the infrastructure impl at runtime. The application never names the implementation class. |
| Single `service/` package containing both `OrderApplicationService` and `OrderDomainService` | Hides the layer distinction. Reviewers can't tell at a glance which class owns transactions and which owns rules. | Split into `application/` and `domain/`. The two have different jobs and different layer constraints. |
| JAX-RS resource calling a repository directly (`@Inject PaymentRepository`) | Skips the application service. Transaction boundary, DTO mapping, and use-case logic happen in the wrong layer. | Resource depends only on `application`. Application service owns the orchestration. |
| One package per technical kind (`controller/`, `service/`, `repository/`, `entity/`) at the bounded-context root | This is "horizontal slicing" — it scatters one feature across four packages and makes layer violations invisible. The DDD layout is "vertical": one BC, four layers. | Adopt the four-layer structure. Each BC is self-contained vertically. |

---

## Excuse / Reality

When you catch yourself reasoning around the rules above, look here before you type. The left column is verbatim — what you'll actually say in your head or in Slack. The right column is what defeats it.

| Excuse | Reality |
|---|---|
| "Repository = persistence = infrastructure. The interface goes next to its implementation; that's how every Spring tutorial does it." | Tutorial-as-authority is the cargo-cult version of architecture. The interface describes a domain contract — what the aggregate needs to be loaded and saved — in domain language. The implementation supplies persistence. Putting the interface in infrastructure forces the domain to import infrastructure to refer to its own contract. The contract belongs with the thing it serves, not the thing that fulfills it. |
| "Putting the interface next to its implementation feels tidy and reduces folder navigation." | "Tidy" is comparing on the wrong axis. The dependency direction is the architectural axis; folder ergonomics is the personal-preference axis. When the two conflict, the dependency direction wins, because broken dependencies show up as runtime cycles or framework coupling — not as inconvenience. |
| "It's just a service — `service/` is fine for both `ApplicationService` and `DomainService`." | The two have different jobs: orchestration owns transactions and DTOs; domain logic owns invariants and is pure. Lumping them in one folder is the layer-confusion smell that lets transaction logic creep into domain code (and vice versa). The qualifier in the class name *and* the layer in the package together prevent the drift. |
| "We're a small team / small BC — let's keep it flat now and split when it grows." | Dependency violations don't show up in a flat layout because there are no boundaries to enforce them. Flat-now-split-later means the splitting work has to disentangle violations that accumulated. The moment you grow past three classes, the layering map costs less than the cleanup. |
| "The framework wants the entity to extend `PanacheEntity` — the active-record shape is the framework-suggested one, so the aggregate has to carry the JPA annotations." | The DDD-foundations rule is "domain depends on nothing in your code" — and that includes `jakarta.persistence.*`. The aggregate is a plain POJO in `<bc>.domain`; the JPA entity is a *separate* class in `<bc>.infrastructure`; the repository implementation translates. Panache's active-record shape is for *persistence types* — perfectly fine on the infrastructure-layer mirror class. Conflating the domain aggregate and the JPA entity into one type is a Quarkus-mechanics shortcut for CRUD apps, not a license to ship ORM concerns into the domain layer. (Detailed in `ddd-aggregates`; the persistence-side mechanics live in `quarkus-persistence`.) |

---

## Quick Reference

### Class type → package

| Class kind | Package |
|---|---|
| Entity | `<bc>.domain` |
| Value object | `<bc>.domain` |
| Aggregate root | `<bc>.domain` |
| Repository **interface** | `<bc>.domain` |
| Domain service (stateless, framework-free) | `<bc>.domain` |
| Domain event (record) | `<bc>.domain` |
| Application service (use-case orchestration) | `<bc>.application` |
| Command DTO (input) | `<bc>.application` |
| Read DTO / query result | `<bc>.application` |
| Repository **implementation** (Panache, JDBC) | `<bc>.infrastructure` |
| External-system adapter (HTTP client, Kafka producer/consumer) | `<bc>.infrastructure` |
| JAX-RS resource | `<bc>.interfaces.rest` |
| `ExceptionMapper` | `<bc>.interfaces.rest` |

### Class name → suffix

| Class kind | Suffix |
|---|---|
| Application service | `*ApplicationService` |
| Domain service | `*DomainService` |
| Aggregate root, entity, value object | (no suffix — name is the noun: `Order`, `Payment`, `Money`) |
| Repository interface | `*Repository` |
| Repository implementation | `Panache*Repository` (or `*RepositoryImpl` if the impl detail isn't worth highlighting) |
| Domain event | `*Event` (past tense — `OrderPlacedEvent`, never `PlaceOrderEvent`) |

### Allowed dependency directions

```
interfaces.rest ──▶ application ──▶ domain
                                       ▲
                            infrastructure
```

Read it as: arrows go from *who knows about whom*. Domain knows about nothing. Application knows about domain.
Infrastructure knows about domain (to implement its interfaces). Interfaces knows about application (to call its
services). Any other arrow is a layering violation.

### Companion skills

- **`ddd-aggregates`** — aggregate root invariants, identity strategy, factories, cross-aggregate references by ID.
- **`ddd-value-objects`** — record-based value objects, equality by value, validation in canonical constructors.
- **`ddd-services`** — application service vs domain service, in depth.
- **`ddd-repositories`** — repository interface design and the rules about returning aggregates.
- **`ddd-domain-events`** — past-tense naming, raise vs dispatch, CDI `Event<T>` with `AFTER_SUCCESS`.
- **`quarkus-persistence`** / **`quarkus-rest`** / **`quarkus-testing`** / **`quarkus-logging`** — the
  Quarkus mechanics inside each layer.

---
> Source: [jeremyrdavis/ddd-foundations](https://github.com/jeremyrdavis/ddd-foundations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
