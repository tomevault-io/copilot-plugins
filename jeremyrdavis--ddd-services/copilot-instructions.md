## ddd-services

> >


# DDD Services Skill

Two kinds of services, two layers, two jobs. **Application services orchestrate** (load, mutate, persist,
dispatch — own the transaction). **Domain services compute** (pure logic over domain inputs, no I/O, no
framework). The class-name suffix and the package must agree.

**Foundational principle.** Aggregates guard invariants. Domain services compute across inputs. Application
services orchestrate use cases. Each one belongs in a different layer with different dependencies allowed. A
class named `*DomainService` that injects `@Transactional` and a repository is an application service wearing
the wrong hat — the suffix is a label, not a license. A class named just `*Service` (no qualifier) is the
layer-confusion smell that lets transaction logic creep into pure rules and pure rules creep into orchestration.

**Red Flags — STOP if you find yourself thinking:**

- About to name a class `OrderService` (no `Application` or `Domain` qualifier) — pick one and put it in the
  matching package.
- About to put `@Transactional` on a class named `*DomainService`.
- About to `@Inject` a repository, an `Event<>`, an HTTP client, a Kafka producer, or any other I/O thing into
  a `*DomainService`.
- About to put a `*DomainService` in `<bc>.application` (or a `*ApplicationService` in `<bc>.domain`).
- About to inline 50+ lines of business rules as a `private` method inside an application service.
- About to put a multi-line policy calculation inside an aggregate root method that should just enforce a state
  transition (e.g. an 80-line refund-amount calculation inside `Order.refund(...)`).
- About to make a domain service `static` to "avoid the CDI ceremony" — domain services are CDI-managed
  `@ApplicationScoped` classes; the discipline is *what's injected into them* (nothing framework-bound), not
  whether they're managed.

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new service class for any use case.
- Naming a class that ends in "Service" — picking `*ApplicationService` vs `*DomainService` vs neither (and
  putting the logic on the aggregate instead).
- Reviewing a diff with a `*DomainService` that has `@Transactional` or an injected repository.
- Reviewing a diff where business rules are inlined as a private method in an application service (often
  signals a missing domain service).
- Reviewing a diff where an aggregate root method computes policy in addition to enforcing the state
  transition.

**Out of scope**: layering and package placement (use `ddd-foundations` — this skill assumes the
`<bc>.application` and `<bc>.domain` packages and dependency directions); aggregate-root invariants and state
machines (use `ddd-aggregates`); repository interface design (use `ddd-repositories`); domain-event payload and
dispatch mechanics (use `ddd-domain-events` — this skill mentions that the application service drains and
dispatches, but the *event shape and observer rules* live there); the Quarkus mechanics of `@Transactional`
placement and CDI scopes (use `quarkus-persistence`).

---

## Core Rules

1. **Two kinds of services, two suffixes, two layers.** `*ApplicationService` lives in `<bc>.application`.
   `*DomainService` lives in `<bc>.domain`. Never name a class just `*Service` — the unqualified name is a
   layer-confusion smell.
2. **Application service = use-case orchestration.** Loads aggregates via repository interfaces, calls domain
   methods on them, optionally calls a domain service for cross-aggregate computation, persists via the
   repository, drains and dispatches `pendingEvents`, maps the result to a DTO, and returns. Owns the
   `@Transactional` boundary (per `quarkus-persistence`).
3. **Application service may inject:** repository interfaces, other application services (sparingly), domain
   services, `Event<DomainEvent>`, `Clock` and other framework-supplied infrastructure. It is the correct
   place to mention `@Transactional`, `Event`, and any I/O bean.
4. **Domain service = pure logic that doesn't naturally belong to one aggregate.** Computes across multiple
   aggregates / values / a `Clock` / domain configuration. Returns a domain value (a value object, a verdict,
   a calculated `Money`). No state changes, no persistence, no I/O.
5. **Domain service may inject:** other domain services and CDI-supplied configuration values that are domain
   data (e.g. a `RefundPolicy` configured for the BC). It must not inject repositories, `Event<>`, HTTP/Kafka
   clients, `EntityManager`, `@Transactional` propagation. The dependency rule from `ddd-foundations` (domain
   depends on nothing) is the same rule expressed at this level.
6. **Default first: put logic on the aggregate.** Most domain logic should live inside the aggregate root, on
   the aggregate's domain methods, where it's adjacent to the state it operates on. Extract a domain service
   only when the logic *doesn't naturally belong to one aggregate* — typically because it spans two aggregates
   or two value objects, or because it needs an external input (a `Clock`, a configured policy) the aggregate
   doesn't carry.
7. **Aggregates guard invariants; domain services compute across inputs.** This is the heuristic for choosing
   between Rule 6 (logic on the aggregate) and a domain service. If the operation is "given the current state
   plus this input, am I allowed to do X?" — that's an aggregate method (the aggregate guards its own
   transition). If the operation is "given these inputs, what is the answer?" — that's a domain service
   (computes a result; doesn't mutate state). The aggregate's `refund(Money amount)` enforces the state
   transition; a `RefundCalculatorDomainService` computes the `Money amount` value the aggregate then
   accepts.
8. **The application service is the only layer that owns `@Transactional`.** Not the aggregate (the aggregate
   doesn't know about transactions), not the domain service (domain services do no I/O), not the resource (the
   resource doesn't manage state).
9. **Domain services are still CDI beans.** Annotate `@ApplicationScoped`. The "framework-free" discipline is
   about *what's injected into them*, not whether they're managed by Quarkus. CDI wiring is fine; the line is
   between "this bean is registered with the container" (allowed) and "this bean's behavior depends on
   `@Transactional` or an injected I/O dependency" (not allowed for a domain service).
10. **Application service returns DTOs, not aggregates.** The DTO crosses the application↔interfaces boundary.
    The aggregate stays in `domain`. (This rule lives both here and in `quarkus-rest`/`quarkus-persistence` —
    in this skill it's the *application service's* responsibility to do the mapping.)
11. **Don't name a class just `*Service`.** Pick the suffix and put it in the layer that matches. The
    unqualified name is the smell that lets transaction handling and rule code drift between files in a shared
    `service/` package (covered as an anti-pattern in `ddd-foundations` and reinforced here).

---

## Canonical Example

The "process refund" use case. Three components, three classes, three different roles.

### The aggregate's domain method (lives in `<bc>.domain`, full design in `ddd-aggregates`)

```java
// com.example.orders.domain.Order — sketch of the relevant method only
public void refund(Money amount) {
    if (status != Status.PLACED && status != Status.FULFILLED) {
        throw new OrderRefundNotAllowedException(new OrderId(id));
    }
    if (amount.amount().compareTo(total.amount()) > 0) {
        throw new IllegalArgumentException("refund amount exceeds order total");
    }
    this.status = Status.REFUNDED;
    this.refundedAmount = amount;
    this.pendingEvents.add(new OrderRefunded(new OrderId(id), amount));
}
```

What this enforces: the *invariant* — the order is in a state that allows refund, the amount is not greater
than the total. The aggregate **does not compute** the refund amount; it accepts the value and verifies the
resulting state is legal.

### The domain service (lives in `<bc>.domain`)

```java
package com.example.orders.domain;

import jakarta.enterprise.context.ApplicationScoped;
import java.time.Instant;

@ApplicationScoped                              // CDI-managed, but no @Transactional, no injected I/O
public class RefundCalculatorDomainService {

    public Money compute(Order order, Instant now) {
        Money base = order.total();
        Money prorated = applyTimeProration(base, order.placedAt(), now);
        Money afterRestocking = applyRestockingFees(prorated, order.lines());
        Money rounded = roundToCurrencyFractionDigits(afterRestocking);
        return rounded;
        // ... 80 lines of pure rules: prorated within 30 days, restocking fees on opened SKUs,
        // currency rounding to default fraction digits with HALF_EVEN, etc.
    }

    private Money applyTimeProration(Money base, Instant placedAt, Instant now)        { /* ... */ }
    private Money applyRestockingFees(Money prorated, List<OrderLine> lines)            { /* ... */ }
    private Money roundToCurrencyFractionDigits(Money m)                                { /* ... */ }
}
```

What this provides: pure computation. Inputs are an `Order` aggregate and a `Clock` value (`Instant`). No
repository, no `@Transactional`, no `Event<>`. Trivially unit-testable with a real `Order` instance and a
fixed `Instant` — no `@QuarkusTest` needed.

### The application service (lives in `<bc>.application`)

```java
package com.example.orders.application;

import com.example.orders.domain.*;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.event.Event;
import jakarta.inject.Inject;
import jakarta.transaction.Transactional;
import java.time.Clock;

@ApplicationScoped
public class ProcessRefundApplicationService {

    @Inject OrderRepository orderRepository;       // domain interface (per ddd-foundations / ddd-repositories)
    @Inject RefundCalculatorDomainService calculator;
    @Inject Event<DomainEvent> events;             // CDI event bus
    @Inject Clock clock;                            // injected so tests can fix time

    @Transactional                                  // the transaction boundary is here, only here
    public RefundDTO refund(OrderId orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));

        Money refundAmount = calculator.compute(order, clock.instant());   // domain-service call
        order.refund(refundAmount);                                          // aggregate state transition
        orderRepository.persist(order);                                      // persist via repository

        order.pullPendingEvents().forEach(events::fire);                     // dispatch domain events

        return RefundDTO.of(order, refundAmount);                            // map to DTO and return
    }
}
```

What this orchestrates: load → compute → mutate → persist → dispatch → map. The application service is the
*only* class in this triangle that mentions `@Transactional`, `Event<>`, or the repository. Each line is one
of the orchestration steps; there is no business-rule computation hidden as a private method, because that
computation belongs in the domain service.

**Why this triangle works:** the three classes have different signatures of allowed dependencies, and the
package each lives in enforces those signatures (per `ddd-foundations`). The aggregate has no dependencies
beyond domain types. The domain service has no I/O dependencies. The application service is the layer where
the framework concerns aggregate. If you find yourself wanting to add `@Transactional` or `@Inject
OrderRepository` to a class in `domain/`, you've discovered that class is actually an application service —
move it.

---

## Anti-patterns

| Don't | Why it's wrong | Fix |
|---|---|---|
| Name a class `OrderService` (no `Application` or `Domain` qualifier) | Hides whether the class orchestrates or computes. Reviewers can't tell at a glance which layer rules apply. The two kinds of logic drift into the same file. | Pick a suffix and a layer: `OrderApplicationService` in `application/` or `OrderDomainService` in `domain/`. |
| `@Transactional` on a `*DomainService` | The suffix says "domain logic, no I/O" — `@Transactional` says "owns a transaction, has I/O." The two contradict; the class is an application service mislabeled. | Remove `@Transactional`. If the method genuinely needs a transaction, it's not a domain service — rename to `*ApplicationService` and move to `<bc>.application`. |
| `@Inject OrderRepository` (or `Event<>`, or any HTTP/Kafka client) inside a `*DomainService` | Same: the dependency is I/O-bound. Domain services don't do I/O. The repository call belongs in the application service that *calls* the domain service. | Remove the injection. The application service loads the aggregate, then passes the aggregate (or the value the calculation needs) into the domain service's pure method. |
| Domain service in the `application` package (or application service in the `domain` package) | Package-name and class-name disagree. The package-level dependency rules from `ddd-foundations` aren't enforceable when the class is in the wrong layer. | Move the file to the matching package. The class name picks the package. |
| 80-line refund calculation inlined as `private Money computeRefundAmount(Order, Instant)` inside an application service | The named business concept ("how do we compute a refund?") is hidden inside an orchestration method. It can't be unit-tested without standing up the application service; it can't be reused; it grows in invisibility. | Extract to a `*DomainService` in `<bc>.domain`. The application service injects and calls it; the calculation is testable with no `@QuarkusTest`. |
| 80-line refund calculation inlined inside the aggregate's `Order.refund(...)` method | The aggregate is now responsible for *computing the policy* in addition to *enforcing the state transition*. Two responsibilities, two reasons to change. The method needs an `Instant` parameter that the aggregate doesn't otherwise care about. | The aggregate accepts the *result* (`Order.refund(Money amount)`); the calculation lives in a `*DomainService`. **Aggregates guard invariants; domain services compute across inputs.** |
| One big `RefundService` containing both the 80-line calculation and the orchestration (load / persist / dispatch / `@Transactional`) | Lumps two layers of concern into one class in some unmarked package. Tests touching the calculation need Quarkus startup; tests touching the orchestration touch the calculation accidentally. | Split into `RefundCalculatorDomainService` (pure, in `domain/`) and `ProcessRefundApplicationService` (orchestration, in `application/`). |
| `*DomainService` declared `public class` with all-`static` methods to "avoid CDI ceremony" | Loses the ability to inject other CDI-supplied domain configuration (`@ConfigProperty`-injected `RefundPolicy`), forces all callers to import the static method, and makes mocking impossible if you ever need it. | `@ApplicationScoped` on the class. The "framework-free" discipline is *what's injected*, not whether the bean is managed. |

---

## Excuse / Reality

| Excuse | Reality |
|---|---|
| "One service is simpler — splitting into Application + Domain feels like ceremony, especially when there's only one caller today." | If the calculation is 80 lines of pure rules, it's a cohesive named concept earning its own class — not boilerplate. Inlining it as a `private` method hides the named business abstraction (refund policy is a thing the business talks about) inside orchestration code. The next caller — second use case, scheduled job, batch import — has nowhere to reuse it from. The split is the first time the concept gets a name; that name pays for itself by the second caller. |
| "It's named `*DomainService`, so it's a domain service — the suffix is the contract." | The suffix is a label. The contract is the *dependency signature*: a domain service takes domain inputs and returns domain values, with no I/O and no transaction. A class named `RefundDomainService` that injects `@Transactional` and `OrderRepository` is an application service with a misleading name. Rename it (`ProcessRefundApplicationService`) or remove the dependencies — pick one, then the suffix and behavior agree. |
| "Why extract — it's just a private method? Private methods are fine." | Private methods inside an application service can't be unit-tested in isolation (they require the parent class to be instantiated, with all its `@Inject` fields), can't be reused by another use case without copy-paste, and hide named business concepts behind orchestration code. The extraction cost is one class declaration; the readability and testability gains compound from the first reuse. |
| "Why not just put the 80-line calculation inside the aggregate's domain method? The aggregate already knows its own state." | Aggregates guard invariants; domain services compute across inputs. `Order.refund()` should enforce *"this order is in a state that allows refund and the amount doesn't exceed total"* — that's the invariant. *Computing* the right refund amount needs an `Instant` (a `Clock` value), a `RefundPolicy`, and pricing rules — those aren't part of the aggregate's invariants. Forcing the aggregate to take an `Instant` parameter for one method is the smell that the calculation belongs elsewhere. |
| "Domain services that don't take infrastructure dependencies feel pointless — they're just functions in a class." | Yes, they're "just functions" — but on domain types, in the domain package, with no framework dependencies. The discipline of keeping them framework-free is exactly what makes them unit-testable without `@QuarkusTest`, reusable across use cases, and stable across infrastructure changes (swap the persistence layer, swap the message bus — the domain service doesn't care). The "function in a class" shape is the intentional shape. |
| "Tutorials show `@Service` everywhere with everything in it — Spring/Quarkus convention is one-`Service`-per-class." | Tutorial conventions optimize for showing one feature, not for separation of concerns. A "Spring `@Service`" example with 200 lines is a teaching aid, not a maintenance pattern. DDD's two-services rule is a maintenance pattern: it makes the layer of every method visible from the file path. Tutorial-as-authority is the cargo-cult version of architecture (see `ddd-foundations` Excuse / Reality). |

---

## Quick Reference

### Decision tree

```
You're about to write a new method. Where does it go?

Q1: Does the method change the state of one aggregate while enforcing
    that aggregate's invariants?
    YES → it's an aggregate method (Order.cancel(), Order.markFulfilled()).
          Lives on the aggregate. Return type often void; throws on
          invariant violation.
    NO  → continue to Q2.

Q2: Does the method compute a domain value (a Money, a verdict, a
    score, a derived quantity) from one or more aggregates plus
    auxiliary inputs (Clock, configured policy, other value objects)?
    YES → it's a domain service method.
          Lives in <bc>.domain on a *DomainService class.
          @ApplicationScoped, but no @Transactional, no injected I/O.
    NO  → continue to Q3.

Q3: Does the method orchestrate a use case end-to-end — load aggregate,
    call its domain method, optionally call a domain service, persist,
    dispatch events, map to DTO?
    YES → it's an application service method.
          Lives in <bc>.application on a *ApplicationService class.
          @Transactional, @ApplicationScoped, may inject anything
          allowed in the application layer (repositories, Event<>,
          Clock, other application services).
    NO  → it's not a service method. Probably belongs in the resource
          (HTTP-shape concerns), in infrastructure (adapter to an
          external system), or doesn't exist yet.
```

### Allowed dependencies — at a glance

| Class kind | May inject | Must not inject | Must not annotate |
|---|---|---|---|
| `*ApplicationService` (in `<bc>.application`) | Domain repository interfaces, `*DomainService`, other `*ApplicationService` (sparingly), `Event<DomainEvent>`, `Clock`, `@ConfigProperty` values, infra adapters via their domain interface | Concrete repository implementations, JAX-RS `Response` types | n/a — `@Transactional` belongs here |
| `*DomainService` (in `<bc>.domain`) | Other `*DomainService`, `@ConfigProperty`-supplied domain configuration | Repositories, `Event<>`, HTTP/Kafka clients, `EntityManager`, JAX-RS types | `@Transactional` |
| Aggregate root (in `<bc>.domain`) | Nothing (passed in as method parameters) | Anything | `@Transactional` |

### Naming and placement

| Class kind | Suffix | Package |
|---|---|---|
| Application service | `*ApplicationService` | `<bc>.application` |
| Domain service | `*DomainService` | `<bc>.domain` |
| (Don't use) | `*Service` (no qualifier) | (n/a — the layer is unclear) |

### Companion skills

- **`ddd-foundations`** — the layering map; the `<bc>.application` and `<bc>.domain` packages and the no-qualifier-`*Service` anti-pattern live there in summary form.
- **`ddd-aggregates`** — what the aggregate's `refund(Money)` method enforces; `pendingEvents` collection.
- **`ddd-value-objects`** — the `Money` value object the calculator computes and the aggregate accepts.
- **`ddd-repositories`** — the `OrderRepository` interface the application service injects.
- **`ddd-domain-events`** — the `OrderRefunded` event shape and the `AFTER_SUCCESS` dispatch rules.
- **`quarkus-persistence`** — `@Transactional` placement, the 409 mapping for `OptimisticLockException`.
- **`quarkus-rest`** — how the application service's `RefundDTO` crosses the wire from the resource.
- **`quarkus-testing`** — pure-JVM unit tests for the domain service (no `@QuarkusTest`); `@QuarkusTest` for the application service.

---
> Source: [jeremyrdavis/ddd-services](https://github.com/jeremyrdavis/ddd-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
