## event-outboxer

> Guidance for Claude Code (claude.ai/code) when working in this repository.

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project overview

`event-outboxer` is an open-source Java library implementing the
**Transactional Outbox** pattern for modern Spring Boot services. It is
an embedded, per-service outbox (not a cross-service shared-DB bridge —
see [ADR-0001](docs/adr/0001-local-embedded-outbox-scope.md)).

Architecture is fully designed before implementation — see
[docs/](docs/) and the 25 ADRs in [docs/adr/](docs/adr/). Treat the
ADRs as the source of truth: if implementation must deviate from an
ADR, amend the ADR in the same PR.

## Stack

- **Java 25 baseline (LTS)** — `maven.compiler.release=25`, enforced by
  `maven-enforcer-plugin` (`requireJavaVersion [25,)`). Virtual threads
  (`event-outboxer.handler-executor.type=virtual`) work natively and
  are pin-free thanks to JEP 491. See
  [ADR-0017](docs/adr/0017-java-25-and-spring-boot-3-5-baseline.md).
- **Spring Boot 3.5.6** pinned via `spring-boot-dependencies` BOM
  (source of truth for Spring, Jackson, Micrometer, SLF4J, Logback
  versions).
- **Maven 3.9+**.
- PostgreSQL 15+, KeyDB 6 / Redis 7 (adapter-level).
- JUnit 5 + AssertJ + Testcontainers for testing.

## Module layout (19 modules)

```
event-outboxer (parent pom)
├── event-outboxer-bom                     versions-only BOM for consumers
├── event-outboxer-api                     public API: publisher, handler, listener, domain, exceptions
├── event-outboxer-spi                     ports for adapters (EventStore, WorkerRegistry, ...)
├── event-outboxer-core                    engine (NO Spring dependency, enforced by banned-deps)
├── event-outboxer-storage-inmemory        TEST infrastructure: contract tests, testkit, @Import test config (ADR-0020)
├── event-outboxer-storage-postgres        PostgreSQL adapter (plain JDBC + ConnectionSupplier SPI)
├── event-outboxer-serializer-jackson      Jackson EventSerializer + JacksonObjectMapperFactory
├── event-outboxer-serializer-protobuf     Protobuf EventSerializer (schema-first Message payloads, ADR-0026)
├── event-outboxer-lock-postgres-advisory  pg_advisory_lock EntityLocker (postgres-advisory opt-out)
├── event-outboxer-lock-postgres-lease     lease-table EntityLocker — PostgreSQL default (ADR-0022)
├── event-outboxer-lock-redis              Redis/KeyDB EntityLocker (Lettuce)
├── event-outboxer-cache-redis             Redis/KeyDB MetricsSnapshotCache (Lettuce)
├── event-outboxer-metrics-micrometer      MicrometerOutboxListener
├── event-outboxer-tracing-otel            OpenTelemetry OutboxTracer (ADR-0023)
├── event-outboxer-tracing-micrometer      Micrometer Tracing OutboxTracer (ADR-0023)
├── event-outboxer-admin-actuator          Actuator endpoint over OutboxAdmin SPI
├── event-outboxer-admin-rest              opt-in REST controller over OutboxAdmin SPI
├── event-outboxer-testkit                 SettableClock, ManualEngine, assertions
└── event-outboxer-spring-boot-starter     autoconfiguration + SmartLifecycle + TX integration
```

Java package layout mirrors modules 1-to-1 under
`io.github.bams22.outboxer.*` — see
[ADR-0016](docs/adr/0016-maven-module-structure.md).

## Commands

Always invoke Maven through the bundled wrapper (`./mvnw`). The wrapper
is pinned to Maven 3.9.12 in `.mvn/wrapper/maven-wrapper.properties`;
it downloads the distribution automatically on first use and caches
it under `~/.m2/wrapper`. Do NOT use a system-installed `mvn` — the
wrapper guarantees everyone (including CI) runs the exact same Maven.

Build (unit tests only):
```
./mvnw -B -ntp clean verify
```

Build with integration tests (requires Docker for Testcontainers):
```
./mvnw -B -ntp clean verify -P it
```

Per-module:
```
./mvnw -pl event-outboxer-core verify
./mvnw -pl event-outboxer-storage-postgres -P it verify
```

Auto-fix formatting (Google Java Format via Spotless; `spotless:check`
runs in the default build):
```
./mvnw -B -ntp spotless:apply
```

Release dry-run (stages to local dir; requires GPG key configured):
```
./mvnw clean deploy -P release -DaltDeploymentRepository=local::file:./stage -DskipTests
```

## Invariants — do not break

These are enforced by tests, CI and `maven-enforcer-plugin`. Any change
that conflicts with one must be discussed with the user and captured
in a new or amended ADR.

1. **`event-outboxer-core` must not depend on Spring.** Enforced by
   `bannedDependencies` rule. Core uses only `-api`, `-spi`, SLF4J.
2. **Adapters must not depend on `event-outboxer-core`.** Only on
   `-api` and `-spi` plus their target dependency.
3. **`OutboxEventPublisher.publish()` must participate in the caller's
   transaction.** In the PG adapter this is achieved via the
   `ConnectionSupplier` SPI port; the Spring starter wires it to
   `DataSourceUtils.getConnection(ds)` on a
   `TransactionAwareDataSourceProxy`. See
   [ADR-0002](docs/adr/0002-participate-in-client-transaction.md).
4. **Payload is always an explicit DTO.** No lambda serialization (ASM
   tricks), no method references. See
   [ADR-0003](docs/adr/0003-explicit-dto-payload.md).
5. **Handlers must be idempotent.** We offer at-least-once. See
   [ADR-0015](docs/adr/0015-at-least-once-semantics.md).
6. **Optimistic locking via `version`.** Every state-changing operation
   increments it; finalize operations check
   `WHERE version = :claimedVersion AND claimed_by = :workerId`. See
   [ADR-0014](docs/adr/0014-optimistic-locking-via-version-field.md).
7. **Per-event-type isolation.** Each `EventHandler` gets its own
   `ThreadPoolTaskExecutor`, its own poller, its own claim query. See
   [ADR-0004](docs/adr/0004-per-event-type-worker-isolation.md).
8. **Heartbeat lives in a separate `event_outboxer.workers` table** — O(1) per
   JVM, not per in-flight event. See
   [ADR-0005](docs/adr/0005-workers-heartbeat-table.md).
9. **Spring appears only in `event-outboxer-spring-boot-starter` and
   the two admin surface modules (`-admin-actuator`, `-admin-rest`)** —
   autoconfiguration, TransactionAwareDataSourceProxy,
   `ThreadPoolTaskExecutor` with `ContextPropagatingTaskDecorator`;
   the admin modules are Spring-integration surfaces by nature
   (ADR-0019). Core and storage/lock adapters stay Spring-free. See
   [ADR-0009](docs/adr/0009-spring-task-executor-in-starter.md).

## Code style

- **Multi-line Javadoc only.** Never `/** Single line. */`. Always:
  ```java
  /**
   * Description.
   */
  ```
- **Lombok allowed**: `@Builder`, `@AllArgsConstructor`,
  `@NoArgsConstructor`, `@RequiredArgsConstructor` — prefer these over
  hand-written constructors/builders.
- **Google Java Format** via Spotless (`mvn spotless:apply` to fix).
- **Multi-line pom element values** (description, etc.) are expected —
  Spotless `sortPom` formats them.
- **Apache-2.0 header** at the top of every `.java` file (convention
  aligned with Spring/jobrunr/db-scheduler); only `.java`, not xml/yml/
  md/sql.

## Nullness conventions (JSpecify, ADR-0018)

- Every public package has a `package-info.java` with
  `@org.jspecify.annotations.NullMarked` — references are non-null by
  default inside that package.
- Use `@org.jspecify.annotations.Nullable` at the type-use site only
  where a value can legitimately be `null` from a consumer's
  perspective.
- Do NOT add `@NonNull` — it is the default inside `@NullMarked`.
- The dependency is declared as compile-scope in `event-outboxer-api`
  and is transitive to downstream users.
- `Objects.requireNonNull(...)` in compact constructors stays — the
  annotations are static guidance, not runtime validation.

## Where to find things

- **Architecture overview**: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- **All application.yml properties**:
  [docs/CONFIGURATION.md](docs/CONFIGURATION.md)
- **PostgreSQL schema and SQL**: [docs/STORAGE.md](docs/STORAGE.md)
- **Terminology**: [docs/GLOSSARY.md](docs/GLOSSARY.md)
- **Rationale for every design decision**:
  [docs/adr/README.md](docs/adr/README.md) (25 ADRs)
- **Implementation roadmap (phases P0–P10)**: see the plan file noted
  in the user's plan tooling.

## Scope — what this library is NOT

- NOT a cross-service messaging bridge. For that, use a broker (Kafka,
  RabbitMQ) and have the user's `EventHandler` publish to it.
  [ADR-0001](docs/adr/0001-local-embedded-outbox-scope.md).
- NOT exactly-once. It is at-least-once; handler idempotency is the
  user's responsibility.
  [ADR-0015](docs/adr/0015-at-least-once-semantics.md).
- NOT a standalone job scheduler à la jobrunr — no lambda capture, no
  dashboard (in MVP), no cron (in MVP).

## Publication

Artifacts publish to Maven Central under `groupId = io.github.bams22`
via the Sonatype Central Publishing portal (new path, not legacy
OSSRH). GPG signing is required on release.

---
> Source: [bams22/event-outboxer](https://github.com/bams22/event-outboxer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
