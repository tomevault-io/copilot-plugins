## ddd-value-objects

> >


# DDD Value Objects Skill

How to model value objects in a Quarkus DDD project. A value object has no identity, is immutable, equals by
value, and encodes its invariants in its construction. Records are the canonical shape.

**Foundational principle.** A value object encodes its invariants in its constructor. If validation lives
anywhere except the type's constructor, the type is just a container for primitives wearing a class label.
Records are the canonical shape: immutable, value-based equality, validation in the canonical constructor,
replacement (not mutation) via new construction. **Value objects are plain Java records — no
`jakarta.persistence.*` imports, no `@Embeddable`, no `@Id`.** Domain depends on nothing in your code (per
`ddd-foundations`); the persistence layer maps value-object records to columns or to its own `@Embeddable`
mirror class in `<bc>.infrastructure` (covered by `ddd-repositories`). The named failure mode this skill
prevents is **primitive obsession** — modeling a domain concept (money, email, postal address, order ID) as
raw `BigDecimal` / `String` / `Long` fields scattered across the aggregate.

**Red Flags — STOP if you find yourself thinking:**

- About to declare `BigDecimal amount` and `String currency` (or `Currency currency`) as **separate** fields on
  an entity — that's a `Money` value object trying to be born.
- About to use raw `String email`, `String sku`, `String phoneNumber`, or `String addressLine1` on a domain
  class — that's `EmailAddress`, `Sku`, `PhoneNumber`, `AddressLine1` (or just `Address`) trying to be born.
- About to write a JavaBean class with setters and call it a "value object."
- About to hand-write `@Override boolean equals(Object)`, `int hashCode()`, `String toString()` — records do
  this for free.
- About to add a `*Validator` companion class to validate a value-shaped type.
- About to add a method that mutates the value's fields (`this.amount = this.amount.add(...)`).
- About to put `@Data` (Lombok) on a value-object class.
- About to add `@Id` to a value-object record — that gives it identity, which makes it an entity.
- About to write `import jakarta.persistence.*` (or any JPA annotation) on a value-object record in `<bc>.domain`.
- "I'll mark this record `@Embeddable` so the persistence layer can pick it up directly — saves a mapper."
- "JPA needs a no-arg constructor and setters, so records are awkward."
- "BigDecimal and String are simpler — a wrapper is over-engineering."

If any of these surface, re-read Core Rules and Excuse / Reality before typing.

---

## When to Use

- Designing a new value object: `Money`, `EmailAddress`, `Address`, `PhoneNumber`, `IpAddress`, `Coordinates`.
- Designing a typed ID wrapper used as a value at API surfaces: `OrderId(UUID)`, `CustomerId(UUID)`,
  `Sku(String)`.
- Adding behavior to a value object (`Money.plus`, `Money.times`, `EmailAddress.domain`).
- Reviewing a diff with raw primitives where a value object would cohere (`BigDecimal amount` + `String currency`),
  hand-written `equals`/`hashCode` on a value-shaped class, a `*Validator` companion for a `Money`-shaped type,
  mutating methods on a class that should be immutable, or **`jakarta.persistence.*` imports on a value-object
  record in `<bc>.domain`** (the value object is a plain record; mapping is the persistence layer's job).

**Out of scope**: layering and package placement (use `ddd-foundations`); aggregate-root design (use
`ddd-aggregates` — it covers identity, factory methods, state machines); the persistence-layer mapping that
turns a value-object record into columns, or into a sibling `@Embeddable` mirror class in `<bc>.infrastructure`
(use `ddd-repositories`); domain-event payload conventions (use `ddd-domain-events` — events are records but
their *naming and dispatch* live there); the Quarkus mechanics of `@Embeddable` records on Hibernate (covered
by `quarkus-persistence`); domain services (use `ddd-services`).

---

## Core Rules

1. **A value object is a Java record.** That gives you immutability, value-based `equals`/`hashCode`/`toString`,
   and a compact declaration without boilerplate. Don't write a class for a value object unless you have a
   measured reason the record can't satisfy.
2. **Validate in the canonical (compact) constructor.** `public Money { Objects.requireNonNull(currency); if
   (amount.signum() < 0) throw ...; }` — the validation runs every time the record is constructed, including
   when Hibernate hydrates it from a row. There is no "valid path" and "invalid path"; there is one path.
3. **Normalize in the canonical constructor by reassigning the parameter.** Records allow `value = value.trim()
   .toLowerCase()` inside the canonical constructor — the canonical-constructor reassignment is the idiom for
   trimming, lowercasing, scaling decimals, etc.
4. **Equality is by value.** Records produce equals/hashCode that compare every component. Don't override them.
   If two value objects with the same components must compare *unequal*, you have an entity (it has identity);
   use the aggregate-root pattern (covered by `ddd-aggregates`) instead.
5. **Value objects are immutable.** No setters. No methods that change `this`. Operations that "modify" the
   value return a *new* value: `Money.plus(other)` returns a new `Money`, never mutates the receiver. Records
   enforce this — fields are `final`.
6. **The value-object record is a plain Java record.** No `jakarta.persistence.*` imports. No `@Embeddable`.
   No `@Id`. Domain depends on nothing in your code (per `ddd-foundations`); the persistence layer maps the
   value-object record to columns or to a sibling `@Embeddable` mirror class in `<bc>.infrastructure`. The
   mapping responsibility belongs to the repository implementation (covered by `ddd-repositories`), not to the
   record. Hibernate ORM 6 supports records as `@Embeddable` natively — that capability lives in the
   persistence-layer mirror class, not on the domain record.
7. **Never put `@Id` on a value object.** `@Id` introduces identity; identity makes it an entity, not a value.
   If you find yourself wanting to identify a value object across rows, you don't have a value object — you
   have an entity (use `ddd-aggregates`).
8. **Behavior lives on the value object, not in a service.** `Money.plus`, `Money.times`, `EmailAddress
   .domain()` — the operations on a value belong in the value's own type. A `MoneyService.add(BigDecimal,
   BigDecimal, String, String)` taking primitives is the smell that the type should have been a value object.
9. **Typed ID wrappers are value objects.** `record OrderId(UUID value) { public OrderId { Objects.requireNonNull
   (value); } }`. Same rules: record, validation in canonical constructor, equality by value. The wrapper makes
   `placeOrder(OrderId, CustomerId)` impossible to call with arguments swapped — *that* is the value the
   wrapper provides.
10. **Don't use Lombok `@Value` or `@Data` on a value object.** Records already give you what `@Value` was
    designed for — without an annotation processor, without IDE plugin requirements. `@Data` is actively wrong:
    it generates setters and `equals`/`hashCode` over every field including the id, which is the opposite of
    what a value object needs.
11. **The named failure mode is *primitive obsession*.** When you find yourself writing `BigDecimal totalAmount;
    Currency currency;` as two separate fields, or `String email;`, or threading `(BigDecimal, String)` pairs
    through method signatures — you have an unborn value object. The fix is always the same: introduce the
    record.

---

## Canonical Example

Two value objects (`Money`, `EmailAddress`), two typed ID wrappers (`OrderId`, `CustomerId`), and a sketch of
their use in two entities.

```java
package com.example.shared.domain;

import java.math.BigDecimal;
import java.util.Currency;
import java.util.Objects;

public record Money(BigDecimal amount, Currency currency) {

    public Money {
        Objects.requireNonNull(amount, "amount required");
        Objects.requireNonNull(currency, "currency required");
        if (amount.signum() < 0) {
            throw new IllegalArgumentException("amount must be non-negative");
        }
        // Normalize the scale to the currency's default fraction digits, banker's rounding.
        amount = amount.setScale(currency.getDefaultFractionDigits(), java.math.RoundingMode.HALF_EVEN);
    }

    public static Money zero(Currency currency) {
        return new Money(BigDecimal.ZERO, currency);
    }

    public Money plus(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("currency mismatch: " + currency + " vs " + other.currency);
        }
        return new Money(amount.add(other.amount), currency);
    }

    public Money times(int quantity) {
        if (quantity < 0) {
            throw new IllegalArgumentException("quantity must be non-negative");
        }
        return new Money(amount.multiply(BigDecimal.valueOf(quantity)), currency);
    }
}
```

```java
package com.example.shared.domain;

import java.util.Objects;
import java.util.regex.Pattern;

public record EmailAddress(String value) {

    private static final Pattern PATTERN = Pattern.compile("^[^@\\s]+@[^@\\s]+\\.[^@\\s]+$");

    public EmailAddress {
        Objects.requireNonNull(value, "email required");
        // Normalize via canonical-ctor reassignment: trim + lowercase.
        value = value.trim().toLowerCase();
        if (!PATTERN.matcher(value).matches()) {
            throw new IllegalArgumentException("invalid email: " + value);
        }
    }

    public String domain() {
        return value.substring(value.indexOf('@') + 1);
    }
}
```

```java
package com.example.orders.domain;

import java.util.Objects;
import java.util.UUID;

// Typed ID wrapper — same rules: record, canonical-ctor validation, equality by value.
public record OrderId(UUID value) {
    public OrderId {
        Objects.requireNonNull(value);
    }
    public static OrderId fresh() {
        return new OrderId(UUID.randomUUID());
    }
}

public record CustomerId(UUID value) {
    public CustomerId {
        Objects.requireNonNull(value);
    }
}
```

How they're used in two domain aggregates (sketches — full aggregate-root rules in `ddd-aggregates`).
**Plain POJOs — no JPA. Persistence happens in a separate file.**

```java
package com.example.orders.domain;

public class Order {
    private final UUID id;
    private long version;
    private final CustomerId customerId;     // typed ID wrapper as a domain field
    private Money total;                     // value object held as a plain field
    // ... static place(...) factory, cancel() / markFulfilled() domain methods, etc.
}

package com.example.customers.domain;

public class Customer {
    private final UUID id;
    private long version;
    private EmailAddress email;              // value object held as a plain field
    private String name;
    // ... static register(...) factory, etc.
}
```

How they're persisted (sketch — full mapping rules in `ddd-repositories`). The persistence-layer JPA entity
in `<bc>.infrastructure` is what carries `@Embedded` (or split columns); the repository implementation
translates between domain `Money` ↔ JPA columns.

```java
// com.example.orders.infrastructure.persistence
@Entity
@Table(name = "orders")
public class OrderJpaEntity extends PanacheEntityBase {
    @Id public UUID id;
    @Version public Long version;
    public UUID customerId;
    @Embedded public MoneyJpa total;          // JPA-friendly @Embeddable mirror
    // ...
}

@Embeddable
public record MoneyJpa(BigDecimal amount, String currencyCode) {
    public Money toDomain() { return new Money(amount, Currency.getInstance(currencyCode)); }
    public static MoneyJpa from(Money m) { return new MoneyJpa(m.amount(), m.currency().getCurrencyCode()); }
}
```

Or, if the persistence team prefers, the JPA entity can hold flat columns (`BigDecimal totalAmount; String
totalCurrency;`) and the repository assembles the `Money` directly. Either is fine — it's a persistence-layer
choice. What matters for *this* skill is: the domain `Money` record knows nothing about either approach.

What this demonstrates:

- **Records (Rule 1):** `Money`, `EmailAddress`, `OrderId`, `CustomerId` are all records. Equality, hashCode, and
  toString are generated. No setter, no manual `equals`.
- **Validation in the canonical constructor (Rule 2):** every record validates its components in the compact
  constructor. The validation runs whether the record is built by application code or hydrated by Hibernate.
- **Normalization via reassignment (Rule 3):** `EmailAddress` lower-cases and trims the email; `Money` sets the
  scale to the currency's default fraction digits with banker's rounding. The compact-constructor reassignment
  idiom is the right place for this.
- **Behavior on the value (Rule 8):** `Money.plus`, `Money.times`, `EmailAddress.domain()`. The operations
  belong to the value, not to a `MoneyService` taking primitives.
- **Replacement, not mutation (Rule 5):** `Money.plus(other)` returns a new `Money`. The receiver is never
  modified.
- **Plain records, no JPA imports (Rule 6):** `Money`, `EmailAddress`, `OrderId`, `CustomerId` have zero
  `jakarta.persistence` imports. The persistence-layer `OrderJpaEntity` (in `<bc>.infrastructure`) is the
  class that carries `@Embedded` / `@Embeddable` for the `Money` mapping. Aggregates hold value objects as
  plain fields (per `ddd-aggregates`). No `@Id` anywhere on these records — they're values, not entities
  (Rule 7).
- **Typed ID wrappers (Rule 9):** `OrderId(UUID)` and `CustomerId(UUID)` make argument-swap mistakes a compile
  error. The factory `OrderId.fresh()` is a convenience; not required.

---

## Anti-patterns

| Don't | Why it's wrong | Fix |
|---|---|---|
| `public BigDecimal totalAmount; public String currency;` (two fields on the aggregate) | Primitive obsession. The two fields are conceptually one value (`Money`), and the type system has no way to enforce that they're consistent or that `totalAmount` is non-negative. | Plain `record Money(BigDecimal amount, Currency currency)` in `<bc>.domain` with validation in the canonical constructor; aggregate holds `private Money total;` as a plain field. |
| `public String email;` on `Customer` | Same shape. Every code path that touches `email` re-implements (or forgets) the format check, the trim, the lowercasing. | Plain `record EmailAddress(String value)` with normalization + format check in the canonical constructor. |
| JavaBean class with `public Money() {}`, setters, and hand-written `equals`/`hashCode` | Records exist for exactly this. The class form is mutable (a setter call elsewhere can break the value), and the manual equals will drift from the field set on the next `git pull`. | Convert to a plain Java record. Delete the manually-written `equals`/`hashCode`/`toString`. |
| `@Embeddable` (or any other `jakarta.persistence.*` annotation) on a value-object record in `<bc>.domain` | Domain depends on `jakarta.persistence`. The value object's design is now constrained by what Hibernate can map. | Drop the `@Embeddable` from the record. The persistence-layer JPA entity in `<bc>.infrastructure` is the one that carries `@Embeddable` / `@Embedded` (or maps to flat columns) — covered by `ddd-repositories`. |
| `MoneyService.add(BigDecimal a, BigDecimal b, String currencyA, String currencyB)` | The method takes primitives because the value type doesn't exist. The "service" is just a container for behavior the value object should own. | Define `Money` as a record; move `add` to the record as `Money.plus(Money other)`. Delete `MoneyService`. |
| `EmailValidator.validate(String email)` called from the application service before constructing `Customer` | Validation lives outside the type. Anyone forgetting to call the validator gets an invalid `Customer` past the boundary. | Move validation into `EmailAddress`'s canonical constructor. Caller-forgets-to-call becomes "the type cannot be constructed invalid." |
| `public void add(Money other) { this.amount = this.amount.add(other.amount); }` (mutating method) | Value objects must be immutable. A mutating "value object" breaks the value contract — two equals-but-aliased instances become unequal after one is mutated, and consumers that cached a hashcode get a stale bucket. | Make the operation return a new value: `public Money plus(Money other) { return new Money(amount.add(other.amount), currency); }`. |
| `@Data` (Lombok) on a value-object class | `@Data` generates setters (defeats immutability) and an equals over every field that drifts from intent. | Use a plain record. Records are immutable and generate value-based `equals`/`hashCode`/`toString` automatically; no annotation processor required. |
| `@Id` on a value-object record (or any class meant to be a value object) | Identity makes it an entity, not a value. The two have different equality semantics and different lifecycle. | If the type genuinely has identity, it's an aggregate root — apply `ddd-aggregates`. If it doesn't, drop the `@Id` (and any other `jakarta.persistence` import). |
| Fields typed as raw `UUID` / `Long` on method signatures (`placeOrder(UUID id, UUID customerId)`) | Two `UUID`s in a row at a call site is an argument-swap waiting to happen. Compiler can't tell `OrderId` from `CustomerId` if both are `UUID`. | Typed wrappers: `placeOrder(OrderId id, CustomerId customerId)`. The wrapper makes the swap a compile error. |

---

## Excuse / Reality

When you catch yourself reasoning around the rules above, look here before you type. The left column is verbatim — what you'll actually say in your head or in Slack. The right column is what defeats it.

| Excuse | Reality |
|---|---|
| "`BigDecimal` and `String` are simpler — a wrapper is over-engineering. The validation can live in a service." | This is *primitive obsession*, named exactly. Validation that lives somewhere else than the type is validation that will be bypassed — every caller that forgets to invoke the validator gets a `BigDecimal totalAmount = -50` past the boundary. The wrapper is not over-engineering; it's the only way the type system can enforce what the comment says. Simplicity at the field declaration becomes complexity-by-vigilance everywhere else. |
| "JPA needs a no-arg constructor and setters, so records are awkward / impossible." | The domain record doesn't talk to JPA at all. The persistence-layer mirror (in `<bc>.infrastructure`) handles whatever the framework requires; on Hibernate ORM 6 / Jakarta Persistence 3.1 (Quarkus 3.x) that mirror can itself be an `@Embeddable record` — but that's a persistence-layer detail, not a constraint the domain record has to honor. |
| "I'll just mark this domain record `@Embeddable` so the persistence layer can pick it up directly — saves a mapper class." | The "saved" mapper class is a persistence-layer concern; the cost of "saving" it is making the domain record import `jakarta.persistence`. Once that import exists, the domain depends on the framework — ORM upgrades, mapping-strategy changes, and column-naming policies cascade through the model. The mapper sketch in `ddd-repositories` is 5 lines (`from(Money)` and `toDomain()` static methods on the persistence-layer record); the domain stays clean. |
| "Lombok `@Value` / `@Data` removes the boilerplate without converting to a record." | Records already remove the boilerplate without an annotation processor, IDE plugin, or build-tool plugin. `@Value` is the closest Lombok equivalent and is strictly worse: an annotation that has to be expanded vs a language feature that doesn't. `@Data` on a JPA entity is actively wrong (mutable + equals over the id). Just use the record. |
| "I'll write the value as a class so I can subclass it later if requirements change." | Value objects are not subclassed. Inheritance breaks equality (a `BlackFridayMoney extends Money` is `equals` to a `Money` with the same amount but `hashCode`s might differ). If requirements change, add a *new* value object or extend the existing record's behavior with new methods — don't subclass. |
| "Records can't have validation logic — they're just data carriers." | False. The canonical (compact) constructor runs every time the record is constructed and is the standard place for validation, normalization, and parameter reassignment (trim, lowercase, scale-to-currency). Records can also have static factories, instance methods, and computed fields. The "just data" framing is a misreading. |
| "Adding a typed `OrderId` wrapper around `UUID` is over-engineering — `UUID` already prevents collision." | The wrapper isn't about collision (UUIDs handle that). It's about the *position* of the argument at the call site. `placeOrder(UUID id, UUID customerId)` accepts swapped arguments at no compile-time cost; `placeOrder(OrderId id, CustomerId customerId)` makes the swap a compile error. The cost is one record (3 lines); the benefit is permanent. |

---

## Quick Reference

### Value-object decision tree

```
Has identity (two instances with the same fields are still distinct)?
├── Yes → it's an entity / aggregate root → use ddd-aggregates
└── No  → it's a value object → use the rules in this skill
```

### Value-object commit checklist

When you commit a new value object, every line below should be true:

- [ ] Declared as a `record`, not a `class`.
- [ ] Validation in the canonical (compact) constructor.
- [ ] Normalization (trim, lowercase, scale) via canonical-constructor reassignment.
- [ ] No setters. No mutating methods. Operations return new instances.
- [ ] No hand-written `equals`, `hashCode`, or `toString` (records generate them).
- [ ] No Lombok `@Value` or `@Data`.
- [ ] No `@Id`. (If it has identity, it's not a value object.)
- [ ] No `jakarta.persistence.*` imports on the record. The persistence-layer mapping (flat columns, or a sibling `@Embeddable` mirror class in `<bc>.infrastructure`) is the repository's responsibility, not this record's.
- [ ] If typed ID wrapper: still a record; canonical-constructor validates the underlying value is non-null.
- [ ] Behavior (arithmetic, derivation) lives as instance methods on the record itself.

### Value object vs entity

| You see | It's a |
|---|---|
| Two instances with the same data should compare equal | Value object |
| Two instances with the same data should still be distinguishable | Entity (use `ddd-aggregates`) |
| You want to track changes over time / a state machine on the type | Entity |
| The type is a "frozen" measurement, descriptor, or label | Value object |
| The type has an `@Id` | Entity (by definition; remove the `@Id` if it shouldn't be one) |
| The type is referenced by other types' fields but never identified independently | Value object |

### Companion skills

- **`ddd-foundations`** — the layering map; value objects live in `<bc>.domain` (or `<shared>.domain` if shared
  across BCs).
- **`ddd-aggregates`** — aggregates that own and embed value objects.
- **`ddd-domain-events`** — events are records too; this skill's canonical-constructor rule applies to event
  payloads as well.
- **`ddd-repositories`** — the persistence-layer mapping that turns a value-object record into columns or
  a sibling `@Embeddable` mirror class.
- **`quarkus-persistence`** — Hibernate ORM 6 mechanics for the persistence-layer types that *do* use
  `@Embeddable`: `@AttributeOverride`, column naming for embedded fields, and so on. Those concerns live in
  `<bc>.infrastructure`, not in the domain record.

---
> Source: [jeremyrdavis/ddd-value-objects](https://github.com/jeremyrdavis/ddd-value-objects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
