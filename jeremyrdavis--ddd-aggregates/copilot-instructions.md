## ddd-aggregates

> >


# DDD Aggregates Skill

How to design an aggregate root in a Quarkus DDD project. The aggregate is **a plain POJO** in `<bc>.domain`
that enforces its own invariants and is the boundary of one transaction. The persistence-layer JPA entity is a
*separate* class in `<bc>.infrastructure` (covered by `ddd-repositories`); the aggregate has no knowledge of it.

**Foundational principle.** Domain depends on nothing in your code (per `ddd-foundations`) — and that includes
`jakarta.persistence.*`. The aggregate is a plain POJO with no JPA annotations, no Panache extension, no
`EntityManager` reference. The aggregate enforces its own invariants; anything outside it that "validates"
before mutating is a sign that the invariant doesn't actually live there. The aggregate's public surface is the
set of legal operations; private state plus domain methods is how you guarantee that surface is exhaustive.
Public fields, public setters, external `*Validator` classes, and JPA annotations on the aggregate are
different shapes of the same anti-pattern: they let infrastructure or untrusted code shape the domain layer.

**Red Flags — STOP if you find yourself thinking:**

- About to write `import jakarta.persistence.*` (or any `jakarta.persistence.*` annotation) on a class in
  `<bc>.domain`.
- About to write `extends PanacheEntity` / `extends PanacheEntityBase` on an aggregate.
- About to declare a `public` mutable field on an aggregate ("Panache active-record idiom").
- About to add a `setStatus(...)` (or any other setter) on an aggregate root.
- About to add a separate `*Validator` class that "checks the entity is valid" before persistence.
- About to reference another aggregate root by object (`Customer customer` field) instead of by ID
  (`CustomerId customerId`).
- About to call `new Order(...)` from a service method instead of `Order.place(...)` (or another named factory).
- About to put `@Transactional` on a method *inside* the aggregate — the aggregate is a POJO and doesn't
  know about transactions.
- "Mapping the aggregate to a separate JPA entity duplicates the field list — DRY says one type."
- "Panache's idiom is public fields — why fight the framework?"
- "It's a small / CRUD-shaped service — the rich-aggregate ceremony is over-engineering."

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new aggregate root or modifying an existing one.
- Picking the identity strategy: typed UUID wrapper, externally-assigned identifier, or — only when the ID has
  no domain meaning — a plain `Long`.
- Writing the static factory method that constructs a valid initial state.
- Writing the `rehydrate(...)` static method the persistence layer reconstructs from.
- Deciding whether an invariant lives in the aggregate or somewhere else.
- Choosing how to reference another aggregate root (always by ID).
- Adding a `long version` field for optimistic locking.
- Raising a domain event from inside an aggregate when state changes.
- Reviewing a diff with any `jakarta.persistence.*` import or Panache extension on a class in `<bc>.domain` —
  that's a category error this skill prevents.

**Out of scope**: layering and package placement (use `ddd-foundations`); the internal shape and equality of
value objects, including the typed ID wrappers `OrderId`, `CustomerId` (use `ddd-value-objects`); the
persistence-layer JPA entity that mirrors the aggregate, the repository interface, and the repository
implementation that translates between domain and JPA types (use `ddd-repositories`); domain-event payload
conventions and CDI dispatch (use `ddd-domain-events`); application-service orchestration and `@Transactional`
placement (use `ddd-services` and `quarkus-persistence`); the Quarkus mechanics of `@Embeddable` records and
Panache base classes — *those concerns belong to the persistence layer, not the aggregate* (covered in
`quarkus-persistence`).

---

## Core Rules

1. **The aggregate is a plain POJO.** No `jakarta.persistence.*` imports. No `@Entity`, `@Id`, `@Version`,
   `@Embedded`, `@ElementCollection`, `@Transient`. No `extends PanacheEntity` / `PanacheEntityBase`. No
   `EntityManager` reference. The aggregate lives in `<bc>.domain` and depends on the JDK plus its own domain
   types (per `ddd-foundations`). The persistence-layer JPA entity is a **separate class** in
   `<bc>.infrastructure` and the repository implementation maps between them (covered by `ddd-repositories`).
2. **The aggregate is the only entry point to its internal state.** No public fields, no setters. Mutation
   happens through named domain methods (`cancel()`, `markFulfilled()`, `addLine(...)`) that enforce the
   invariants of that operation.
3. **Private state, public domain methods.** Fields are `private`; expose what callers need via accessor methods
   (`status()`, `total()`) — not via getters that mirror every field. The aggregate decides what's visible.
4. **Construct via a static factory, not `new`.** A method like `Order.place(id, customerId, lines)` validates
   inputs and returns a fully-constructed, valid aggregate. The constructor is `private`; external callers
   never see it.
5. **The factory enforces creation invariants.** Empty line list? Mixed currencies? Missing customer ID? The
   factory throws — it does not return a partially valid aggregate that a downstream `validate()` will repair.
6. **Provide a `rehydrate(...)` static method (or equivalent) for the persistence layer.** The repository
   implementation reconstructs an existing aggregate from stored state via this method. The rehydration path
   trusts that the data was valid when first persisted; it does not re-run creation invariants (no
   `OrderPlaced` event re-emitted, etc). Domain code never calls `rehydrate(...)` — only the persistence layer.
7. **Domain methods enforce transition invariants.** `cancel()` checks the current state and throws if
   cancellation is illegal *before* mutating. Each method states its precondition and either succeeds in full
   or throws — no half-applied state changes.
8. **Reference other aggregates by ID, never by object.** `CustomerId customerId`, not `Customer customer`. Object
   references drag the other aggregate into your transaction (one-aggregate-per-transaction violation), conflate
   identity with reachability, and turn aggregate boundaries into JPA association graphs (in the persistence
   layer, where they belong — but never in the domain).
9. **Use typed ID wrappers when the ID has domain meaning.** `record OrderId(UUID value) {}`. The wrapper makes
   `placeOrder(OrderId, CustomerId)` impossible to call with arguments swapped. Prefer it over raw `UUID`/`Long`
   parameters that are easy to confuse. (Defined in `ddd-value-objects`.)
10. **Carry a plain `long version` field for optimistic locking.** The aggregate stores the version it was
    loaded with; the persistence layer applies the actual `@Version` semantics (in the JPA entity that mirrors
    the aggregate) and translates `OptimisticLockException` → 409 Conflict (per `quarkus-rest`, mapping wired
    in `ddd-repositories`). Skip versioning only when the aggregate is genuinely single-writer.
11. **Hold value objects as plain fields.** `Money total` as a field of type `Money` (a plain record from
    `ddd-value-objects`). No `@Embedded` on the aggregate — `@Embedded` is JPA, lives on the persistence-layer
    entity, never on the aggregate.
12. **Raise domain events from inside the aggregate; dispatch them from the application service.** When
    `cancel()` succeeds it appends an `OrderCancelled` to a `pendingEvents` list on the aggregate. The
    application service drains that list and publishes after the transaction commits (`AFTER_SUCCESS` — see
    `ddd-domain-events`).
13. **One aggregate per transaction.** A `@Transactional` use case loads, mutates, and persists *one*
    aggregate plus its value objects. If two aggregates need to change consistently, the model is wrong —
    use a domain event and `AFTER_SUCCESS` to update the second in a separate transaction.
14. **No `@Transactional` inside an aggregate.** The aggregate is a POJO; it has no annotations from any
    framework. `@Transactional` lives on the application service (covered by `quarkus-persistence` and
    `ddd-services`).

---

## Canonical Example

The `Order` aggregate root in `com.example.orders.domain`. **Plain POJO — zero `jakarta.persistence`
imports, no Panache extension.** Exercises every Core Rule above.

```java
package com.example.orders.domain;

import java.math.BigDecimal;
import java.util.*;

public class Order {

    private final UUID id;                             // identity (Rule 9)
    private long version;                              // optimistic-lock cursor (Rule 10) — plain long, no @Version
    private final CustomerId customerId;               // reference by ID, not by Customer object (Rules 8, 9)
    private final List<OrderLine> lines;
    private Status status;
    private Money total;                               // value object as a plain field (Rule 11)
    private final List<DomainEvent> pendingEvents = new ArrayList<>();

    public enum Status { PLACED, FULFILLED, CANCELLED }

    /** Private — used only by the static factory and rehydrate. */
    private Order(UUID id, CustomerId customerId, List<OrderLine> lines,
                  Status status, Money total, long version) {
        this.id = id;
        this.customerId = customerId;
        this.lines = new ArrayList<>(lines);
        this.status = status;
        this.total = total;
        this.version = version;
    }

    /**
     * Static factory — the only way domain code brings a new Order into existence.
     * Enforces creation invariants (Rule 5) and raises an `OrderPlaced` event (Rule 12).
     */
    public static Order place(OrderId id, CustomerId customerId, List<OrderLine> lines) {
        if (lines == null || lines.isEmpty())
            throw new IllegalArgumentException("an order must have at least one line");
        Currency currency = lines.get(0).unitPrice().currency();
        if (!lines.stream().allMatch(l -> l.unitPrice().currency().equals(currency)))
            throw new IllegalArgumentException("all lines must share a currency");

        Money total = lines.stream()
            .map(OrderLine::lineTotal)
            .reduce(new Money(BigDecimal.ZERO, currency), Money::plus);

        Order order = new Order(id.value(), customerId, lines, Status.PLACED, total, 0L);
        order.pendingEvents.add(new OrderPlaced(id, customerId));   // raise event (Rule 12)
        return order;
    }

    /**
     * Reconstruction path used by the persistence layer (Rule 6). Trusts the inputs
     * already passed validation when first persisted; does not re-emit `OrderPlaced`.
     * Domain code never calls this — only the repository implementation in
     * `<bc>.infrastructure` (covered by `ddd-repositories`).
     */
    public static Order rehydrate(UUID id, CustomerId customerId, List<OrderLine> lines,
                                   Status status, Money total, long version) {
        return new Order(id, customerId, lines, status, total, version);
    }

    /** Transition method — enforces the state-machine invariant (Rule 7). */
    public void cancel() {
        if (status == Status.FULFILLED)
            throw new OrderCancellationNotAllowedException(new OrderId(id));
        this.status = Status.CANCELLED;
        this.pendingEvents.add(new OrderCancelled(new OrderId(id)));
    }

    public void markFulfilled() {
        if (status != Status.PLACED)
            throw new IllegalStateException("only PLACED orders can be fulfilled");
        this.status = Status.FULFILLED;
        this.pendingEvents.add(new OrderFulfilled(new OrderId(id)));
    }

    // Read accessors — only what callers need (Rule 3).
    public OrderId id()            { return new OrderId(id); }
    public Status status()         { return status; }
    public Money total()           { return total; }
    public CustomerId customerId() { return customerId; }
    public List<OrderLine> lines() { return List.copyOf(lines); }   // defensive copy
    public long version()          { return version; }

    /** Drained by the application service after persist; published at AFTER_SUCCESS. */
    public List<DomainEvent> pullPendingEvents() {
        List<DomainEvent> drained = List.copyOf(pendingEvents);
        pendingEvents.clear();
        return drained;
    }
}
```

Companion types (full coverage in `ddd-value-objects` and `ddd-domain-events`) — also plain, also no
persistence:

```java
public record OrderId(UUID value) {
    public OrderId { Objects.requireNonNull(value); }
}

public record CustomerId(UUID value) {
    public CustomerId { Objects.requireNonNull(value); }
}

public record OrderLine(String productSku, int quantity, Money unitPrice) {
    public OrderLine {
        if (quantity <= 0) throw new IllegalArgumentException("quantity must be positive");
    }
    public Money lineTotal() { return unitPrice.times(quantity); }
}

public record OrderPlaced(OrderId orderId, CustomerId customerId)   implements DomainEvent {}
public record OrderCancelled(OrderId orderId)                        implements DomainEvent {}
public record OrderFulfilled(OrderId orderId)                        implements DomainEvent {}
```

### What lives in `<bc>.infrastructure`

The persistence-layer JPA entity that *mirrors* the aggregate is a separate class; the repository
implementation translates between them. Sketch (full coverage in `ddd-repositories`):

```java
// com.example.orders.infrastructure.persistence
@Entity
@Table(name = "orders")
public class OrderJpaEntity extends PanacheEntityBase {
    @Id public UUID id;
    @Version public Long version;
    public UUID customerId;
    @Enumerated(EnumType.STRING) public Order.Status status;
    public BigDecimal totalAmount;
    public String totalCurrency;
    @ElementCollection public List<OrderLineJpa> lines;
    // ... mapping helpers: from(Order) and toDomain() in the repository implementation
}
```

The aggregate has no idea this class exists. The repository implementation imports both, maps fields, and
returns / accepts the aggregate. This is the file separation the layering enforces.

What the canonical example demonstrates:

- **POJO (Rule 1):** zero JPA imports. The class compiles with only the JDK and the BC's own domain types on
  the classpath.
- **No setters, no public mutable fields (Rules 2, 3):** state is `private final` where possible; only
  `version`, `status`, `total`, and `pendingEvents` are mutable, and only domain methods change them.
- **Static factory enforces creation invariants (Rules 4, 5):** empty lines and mixed currencies throw before
  the aggregate exists. No `validate()` method that callers might forget to call.
- **Rehydrate path (Rule 6):** the persistence layer reconstructs an existing aggregate via `rehydrate(...)`;
  no `OrderPlaced` event re-emitted. The domain code never calls `rehydrate(...)`.
- **Cross-aggregate reference by ID (Rule 8):** `CustomerId customerId`, not `Customer customer`. Order doesn't
  drag Customer into anything.
- **Typed identity (Rule 9):** `OrderId(UUID)` and `CustomerId(UUID)` records at the API surface; the internal
  storage is the raw `UUID` to keep the field declaration uncluttered.
- **Plain `long` version (Rule 10):** the persistence-layer entity has `@Version Long version`; the aggregate
  has `private long version`. Same data, different layer.
- **Value object as a plain field (Rule 11):** `Money total` is just a field of type `Money`. No `@Embedded`.
  The persistence-layer entity is the one with `@Embedded` (or with split `totalAmount` / `totalCurrency`
  columns — that's the repository's call).
- **Domain events (Rule 12):** raised inside the aggregate; drained and dispatched by the application service
  (covered by `ddd-services`).

---

## Anti-patterns

| Don't | Why it's wrong | Fix |
|---|---|---|
| `@Entity public class Order extends PanacheEntityBase` (aggregate as a JPA entity) | Domain depends on `jakarta.persistence.*` and Panache. The aggregate's design is now constrained by ORM concerns: protected no-arg constructor for Hibernate, public mutable fields for the Panache active-record idiom, `@ElementCollection` shape pressure on `OrderLine`, eager-fetch surprises, cascade-mode footguns. The "domain depends on nothing" rule from `ddd-foundations` is broken at the source. | The aggregate is a plain POJO in `<bc>.domain`. A separate `OrderJpaEntity` lives in `<bc>.infrastructure` (covered by `ddd-repositories`) with the JPA annotations; the repository implementation maps between them. |
| `@Embedded private Money total;` on the aggregate | `@Embedded` is JPA. The aggregate now imports `jakarta.persistence.Embedded`. Same root violation. | The aggregate holds `private Money total;` — a plain field of a plain record. The persistence-layer entity in infrastructure is the one with `@Embedded` (or with split columns — the repository's call). |
| `@Version public Long version;` on the aggregate | `@Version` is JPA semantics. Domain shouldn't know about the framework's lock detection. | The aggregate carries `private long version;` — a plain `long`. The persistence-layer entity has `@Version`. Equivalent data, one layer wears the annotation. |
| `protected Order() {}` with no callers, "for Hibernate only" | The constructor's only justification is the persistence framework — that's a clear signal the class is being shaped by infrastructure. | The aggregate has a `private` constructor used by the static factory and `rehydrate(...)`. The persistence-layer JPA entity has its own no-arg constructor; that class is a separate file. |
| Public mutable fields on the aggregate ("Panache active-record idiom") | Encapsulation lost; invariants become advisory. The "Panache idiom" argument is a category error — Panache is a persistence-layer convenience, not a domain-modeling guideline. | `private` fields, public domain methods. The Panache shape lives on the persistence-layer entity. |
| `public void setStatus(Status s) { this.status = s; }` | A setter is a domain method with no domain meaning. It accepts any input including ones that violate the state machine. | Replace with named methods: `cancel()`, `markFulfilled()`. The method's name describes the rule. |
| Separate `OrderValidator` class with `validate(Order o)` called by the application service | The invariant is enforced *only if* callers remember the validator. That's the definition of an invariant that isn't one. | Move the validation into the aggregate's factory and domain methods. The aggregate guarantees its own validity. |
| `Customer customer;` field on `Order` (direct object reference between aggregates) | Drags `Customer` into `Order`'s transaction (one-aggregate-per-transaction violation). Lazy-load surprises and cascade-mode footguns follow — and the surprises now live in your domain. | Reference by ID: `CustomerId customerId;`. If you need Customer data, look it up in the application service via the Customer repository. |
| Raw `Long id` parameter on `placeOrder(Long, Long)` (or any pair of raw IDs) | Loses domain-typed identity; makes `placeOrder(Long, Long)` callable with arguments swapped at no compile-time cost. | Typed wrappers at API surfaces: `placeOrder(OrderId id, CustomerId customerId)`. (Defined in `ddd-value-objects`.) |
| `new Order()` from a service, fields then set one by one | Bypasses the factory. The aggregate exists in an invalid intermediate state during the assignments — by the time the last assignment runs, half the invariants are already broken. | Static factory: `Order.place(...)` validates inputs and returns the fully-constructed aggregate. Constructor stays `private`. |
| `@Transactional` annotation on a domain method inside the aggregate | The aggregate is a POJO. Adding `@Transactional` couples it to a framework. | `@Transactional` belongs on the application service method (per `ddd-services` and `quarkus-persistence`). |
| `Order.findById(id)` static call inside the aggregate's domain method | The aggregate is mutating itself by reaching into the persistence layer. Concurrent modification, repository coupling, and a circular dependency all follow. | Aggregate methods take what they need as parameters. The application service does the loading and passes domain values in. |

---

## Excuse / Reality

When you catch yourself reasoning around the rules above, look here before you type. The left column is verbatim — what you'll actually say in your head or in Slack. The right column is what defeats it.

| Excuse | Reality |
|---|---|
| "Quarkus + Panache active record makes public fields idiomatic — `@Entity public class Order extends PanacheEntity` is the framework-suggested shape for any persistent type." | Domain depends on nothing in your code (per `ddd-foundations`) — and that includes `jakarta.persistence.*`. The aggregate is a plain POJO in `<bc>.domain`; the JPA entity is a separate class in `<bc>.infrastructure`; the repository implementation translates. The "framework-suggested shape" is for *persistence types*, not for domain aggregates. Conflating them lets ORM concerns (eager-fetch surprises, `@Transactional` propagation, cascade-mode footguns, no-arg-constructor-for-Hibernate, public-fields-for-Panache) shape your invariant-enforcement layer. The active-record approach is a Quarkus-mechanics shortcut for CRUD apps; for an aggregate with state-machine transitions, cross-aggregate references, and domain events, the cost of conflation eclipses the cost of the mapper. |
| "Mapping the aggregate to a separate JPA entity duplicates the field list — DRY says one type." | The duplication is the cost of separating concerns. The aggregate evolves on domain rules (state-machine changes, new invariants, ubiquitous-language renames); the JPA entity evolves on schema rules (column renames, indexes, denormalization, soft-delete columns). When the two diverge — and they will — having one type forces every change to compromise both. DRY applies to *knowledge*, not to field lists; the two types know different things. The mapping cost is real and bounded; the conflation cost is unbounded and shows up at the worst time. |
| "But Hibernate needs a no-arg constructor — records and POJOs with `private` constructors won't work." | Hibernate needs a no-arg constructor *on the JPA entity it manages*, which is the persistence-layer class — not the aggregate. The aggregate has a `private` constructor used by `Order.place(...)` and `Order.rehydrate(...)`. The JPA entity in `<bc>.infrastructure` has whatever constructor Hibernate requires; that's its problem, not the aggregate's. Two classes, two responsibilities. |
| "Private fields + a static factory + a typed `CustomerId` + a `Money` value object feels like over-engineering for a CRUD-shaped service." | If the requirements list a state-machine transition with a typed exception, a cross-aggregate reference, a multi-field invariant (currency match), and domain events, you don't have a CRUD shape — you have an aggregate. The "ceremony" is load-bearing for every one of those bullets. The over-engineering smell is real for a 3-field lookup table; not real here. |
| "Validation belongs in a separate `Validator` class for testability." | An invariant that's enforced only when the caller remembers to invoke a validator is not enforced. Testability is not improved by moving rules outside the object that owns them — a static factory and a `cancel()` method are trivially unit-testable without a database (no Quarkus, no `@QuarkusTest`, no Dev Services). The validator pattern is a workaround for languages without exceptions and constructors that throw; we have both. |
| "Direct object references between aggregates are simpler than threading IDs through the codebase." | They're simpler at one call site and disastrous everywhere else. A `Customer customer` field on `Order` means: every transaction touching an `Order` also pulls in a `Customer` row; a stale `Customer` from a different transaction can sneak into the lock; the JPA cascade settings now have to be designed across two aggregate boundaries. ID + lookup at the application service is the boring answer that scales. |
| "Setters are how every JPA tutorial models entities — fighting the convention costs more than it saves." | JPA tutorials model *records of data*; DDD aggregates model *invariants*. If the entity is genuinely just a row (a `Country` lookup), setters are fine. If the entity has rules about what state it can be in and which transitions are legal, the setter is a vehicle for breaking those rules silently. Tutorials don't distinguish; you have to. |
| "Adding a `version` field everywhere is paranoid — most updates won't conflict." | The aggregate's `long version` is a one-line cost; the corresponding `@Version` on the persistence-layer entity is the one-line concurrency safety net that turns a silent corruption (last-write-wins) into an `OptimisticLockException` the resource layer can map to 409. Skip it only when the aggregate is genuinely single-writer (e.g. a per-user shopping cart never touched by a background job). For everything else, the field costs less than the incident. |

---

## Quick Reference

### Aggregate-root commit checklist

When you commit a new aggregate root, every line below should be true:

- [ ] Class is a **plain POJO**. Zero `jakarta.persistence.*` imports. Zero `io.quarkus.hibernate.*` imports.
      No `extends PanacheEntity` / `PanacheEntityBase`. No JPA annotations.
- [ ] Class lives in `<bc>.domain` (per `ddd-foundations`).
- [ ] Constructor is `private`; construction is via a static factory (`Order.place(...)`) plus a static
      `rehydrate(...)` for the persistence layer.
- [ ] No `public` mutable fields.
- [ ] No setters. Mutation is through named domain methods.
- [ ] References to other aggregate roots are by ID (typed wrapper from `ddd-value-objects`), not by object.
- [ ] `private long version;` plain field for optimistic locking (the persistence-layer entity has `@Version`).
- [ ] Value objects held as plain fields (`private Money total;`) — no `@Embedded` on the aggregate.
- [ ] Invariants enforced inside the factory (creation) and inside transition methods (mutation).
- [ ] No `*Validator` companion class enforcing rules the aggregate should self-enforce.
- [ ] No `@Transactional` annotation inside the aggregate.
- [ ] Domain events appended to `pendingEvents` from inside the methods that change state.
- [ ] `pullPendingEvents()` (or equivalent) returns and clears the list for the application service to drain.

### Where the persistence-layer entity lives

| Concern | Class | Package | Has JPA annotations? |
|---|---|---|---|
| Domain aggregate (invariants, transitions, events) | `Order` (POJO) | `<bc>.domain` | **No** |
| Persistence-layer entity (table mapping, `@Version`, `@Embedded`) | `OrderJpaEntity` (or similar) | `<bc>.infrastructure` | **Yes** |
| Repository interface (uses domain types) | `OrderRepository` | `<bc>.domain` | **No** |
| Repository implementation (translates aggregate ↔ JPA entity) | `PanacheOrderRepository` | `<bc>.infrastructure` | **Yes** (typically) |

The mapping `Order ↔ OrderJpaEntity` lives in the repository implementation. Covered in `ddd-repositories`.

### Cross-aggregate reference policy

| Need | Do |
|---|---|
| Identify the related aggregate | Hold the typed ID: `CustomerId customerId`. |
| Read data from the related aggregate | Look it up in the application service via its repository; pass the value(s) you need into the aggregate method. |
| Mutate the related aggregate atomically with this one | You can't — and shouldn't. Use a domain event observed at `AFTER_SUCCESS` to mutate the second aggregate in a separate transaction. |
| Display fields from the related aggregate to a client | Map to a DTO in the application service; the resource sees both as DTO fields. |

### Companion skills

- **`ddd-foundations`** — the layering map and the canonical packages where this aggregate lives.
- **`ddd-value-objects`** — the `Money`, `OrderId`, `CustomerId` types this aggregate holds as plain fields.
- **`ddd-repositories`** — the `OrderRepository` interface (in `<bc>.domain`) and the persistence-layer
  `OrderJpaEntity` + repository implementation (in `<bc>.infrastructure`) that translates between domain
  aggregates and JPA entities.
- **`ddd-domain-events`** — what `OrderPlaced` / `OrderCancelled` look like and how the application service
  dispatches them.
- **`quarkus-persistence`** — `@Transactional` placement, optimistic-lock translation to 409, Dev Services.
- **`quarkus-rest`** — how the application service's DTO crosses the wire.
- **`quarkus-testing`** — pure-JVM unit tests for an aggregate's invariants without `@QuarkusTest`.

---
> Source: [jeremyrdavis/ddd-aggregates](https://github.com/jeremyrdavis/ddd-aggregates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
