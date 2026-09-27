## pg-java

> Guidance for AI agents and human contributors working in this repository.

# AGENTS.md

Guidance for AI agents and human contributors working in this repository.
Read this first; it is the contract. `CLAUDE.md` just points here.

## Project overview

`pg-java` is a modern, PostgreSQL-specific database driver for the JVM. The
near-term focus is a clean, idiomatic, PostgreSQL-native API. JDBC compliance is
a long-term goal layered on top of the native API. JDBC must not dictate the
shape of the core driver: keep `java.sql.*` out of `postgresql-client` and
`postgresql-client-protocol`.

## Where things are

Read the relevant doc before changing behavior it governs. Do not re-derive a
decision that already has an ADR.

| Path | What it is |
| --- | --- |
| `docs/adr/` | Numbered, canonical architecture decisions (ADR-0001..). `docs/adr/README.md` is the index. |
| `docs/plans/overall.md` | Master implementation plan. Every numbered item (`C0.1`, `N5.7`, `P4`) is one atomic commit; checkboxes track progress. |
| `docs/follow-up.md` | Index of deferred work, with the reason and the files involved. |
| `docs/plans/*.md` | Per-effort plans (performance, bench module, compat suites). |
| `docs/jdbc-conformance-matrix.md`, `docs/pooler-compatibility.md` | Behavior matrices. |
| `docs/api-surface.md` | The enumerated public API surface (ADR-0020), enforced by `ApiSurfaceManifestTest`. A new public type fails the build until it is listed as Stable or Experimental. |
| `docs/static-analysis.md` | Which analysers the build runs and why, which were rejected, and the scope agreed for the ones not yet adopted. |
| `docs/life-of-a-query.md` | Best single orientation doc for the core execution path. |
| `docs/reviews/`, `docs/benchmarks/` | Findings from past audits; benchmark reference numbers. |
| `compat-suites/` | pgjdbc and Hibernate upstream suites run against our driver, with committed baselines. |
| `scripts/` | Integration matrix, benchmark, and Docker helper scripts. |

ADRs that most often bind a change: ADR-0001 (I/O and concurrency),
ADR-0002 (module boundaries), ADR-0003 (testing strategy), ADR-0004
(compatibility contract / supported servers), ADR-0005 (pull-first results),
ADR-0007 (exceptions), ADR-0012 (prepared statements and query modes).

## Tech stack and requirements

- **Language:** plain Java. No Kotlin, Scala, or other JVM languages.
- **Java 21+.** Prefer records, sealed types, pattern matching, `var`, text
  blocks, enhanced switch where they improve clarity.
- **Virtual threads are first-class.** The I/O layer is blocking-style code run
  on virtual threads, not an async/event-loop framework. Never hold a
  `synchronized` monitor across blocking I/O (use `ReentrantLock`) or virtual
  threads pin to carrier threads.
- **Build:** Apache Maven; `./mvnw` wrapper is checked in. A plain `mvn` must work.
- **Dependencies:** minimal, ideally zero at runtime for the core driver.
  `postgresql-client-protocol` stays dependency-free. Test-only and build-time deps are fine.
  Adding any runtime dependency is a decision to raise explicitly, not to make
  silently (see `docs/dependencies.md`, ADR-0002).

## Module layout

Dependencies flow strictly one way:
`postgresql-client-pgjdbc-compat` -> `postgresql-client-jdbc` -> `postgresql-client` -> `postgresql-client-protocol`.
No reverse or cyclic edges; no `java.sql.*` below `postgresql-client-jdbc`.

- **`postgresql-client-protocol`** - wire protocol encode/decode only. Pure serialization;
  no sockets, connection state, or I/O policy.
- **`postgresql-client`** - the driver: connections, auth (incl. SCRAM), TLS, simple
  and extended query protocols, COPY, LISTEN/NOTIFY, the native public API.
- **`postgresql-client-jdbc`** - the `java.sql.*` adapter on top of core.
- **`postgresql-client-pgjdbc-compat`** - `org.postgresql.*` source-compatibility layer on
  top of the JDBC module. A migration aid, not yet a certified drop-in.
- **`postgresql-client-bench`** (pgbench-style JDBC benchmark), **`postgresql-client-bench-jmh`**
  (server-free micro-benchmarks), **`postgresql-client-coverage`** (aggregate JaCoCo),
  **`postgresql-client-native-smoke`** (GraalVM native-image metadata gate). All
  build-only; never published.

Each shipped module has a `module-info.java`. **A new public package must be
exported there**, or downstream modules fail to compile on the module path.

## Build and test

```sh
./mvnw clean install                 # full build + unit tests + gates
./mvnw verify                        # what CI's unit job runs
./mvnw test                          # unit tests only (no gates, no Docker)
```

Fast inner loops (use these; a full reactor build is rarely what you want):

```sh
./mvnw -q -pl postgresql-client -am test -Dtest=PullResultStreamTest
./mvnw -q -pl postgresql-client -am test -Dtest='Numeric*Test#roundTrips*'
./mvnw -q -pl postgresql-client-jdbc -am -o test    # -o offline once deps are cached
```

- `-pl <module> -am` builds only that module and its upstreams.
- Add `-Dsurefire.failIfNoSpecifiedTests=false` when `-Dtest` targets a class
  that does not exist in every reactor module you built.
- `-T 1C` reactor parallelism is on by default via `.mvn/maven.config`; force a
  serial build with `-T 1` when debugging interleaved output.
- Use `-q -ntp` to keep Maven output readable; read the surefire report at
  `<module>/target/surefire-reports/` rather than scrolling the console dump.

### Integration tests

`*IT` classes run only under the `integration-tests` profile, so the default
build never needs Docker:

```sh
./mvnw verify -Pintegration-tests                          # Testcontainers, postgres:17
./mvnw verify -Pintegration-tests -Dpg.it.image=postgres:14
./mvnw verify -Pintegration-tests -Dpg.it.host=localhost   # use a running server, no Docker
```

`-Dpg.it.host` also takes `pg.it.port` (5432), `pg.it.user`/`pg.it.password`
(postgres), and `pg.it.database` (postgres). The special-purpose harnesses (TLS,
PgBouncer, auth, unix socket) provision their own containers and skip entirely
when `pg.it.host` is set. `scripts/run-integration-matrix.sh` sweeps server
versions; ADR-0004 defines the supported range (9.1-18; PRs gate on 14-18).

### Build gates

Run `./mvnw verify` before declaring a change done. They do not all fire at the
same phase, which is worth knowing when a build fails early: the compiler gates
fail at `compile` and the source-policy tests at `test`, so a plain `mvn test`
already enforces them. Spotless and JaCoCo wait for `verify`, which is what makes
formatting the most common late surprise.

**In the compiler** (configured in the root `pom.xml`, so every build gets them):

- **`-Xlint:all,-this-escape` under `-Werror`**: any javac warning fails the
  build. `this-escape` is the sole exclusion.
- **NullAway at ERROR** over `org.postgresql.client.protocol` and
  `org.postgresql.client.core`: a nullness violation there fails the compile.
  Those prefixes are listed in `NullAway:AnnotatedPackages` in the root POM, so
  everything under them is non-null by default; mark the exceptions with
  jspecify's `@Nullable`. Error Prone is present only as NullAway's carrier, with
  its own checks disabled on purpose -- do not "fix" the `-XepDisableAllChecks`
  flag. See `docs/static-analysis.md`.

**Source-policy tests** (plain unit tests, so they run in the `test` phase):

- **`AsciiSourcePolicyTest`** (in `postgresql-client`): every file under any
  module's `src/`, everything under `docs/`, and the root `README.md` must be
  7-bit ASCII. No em-dashes, curly quotes, or arrows.
- **`NoSynchronizedSourcePolicyTest`** (in `postgresql-client`): the `synchronized`
  keyword is banned outright across the four shipped modules, per ADR-0001. Use
  `ReentrantLock`. A use with genuinely no I/O beneath it goes in that test's
  `ALLOWED` set with its reason, which keeps it a reviewed exception.
- **`ApiSurfaceManifestTest`** (in `postgresql-client`): every public type in an
  exported package must be listed in `docs/api-surface.md` as Stable or
  Experimental.
- **`ModuleLayeringTest`** (in `postgresql-client-jdbc`): `postgresql-client-jdbc` and
  `postgresql-client-pgjdbc-compat` must not import `org.postgresql.client.protocol.*`
  directly; they reach it only transitively through core.

**Later phases:**

- **Enforcer** (`validate`): Java/Maven minimums, dependency convergence, reactor
  convergence.
- **Spotless** (`verify`; `palantirJavaFormat`, 4-space). Fix with
  `./mvnw spotless:apply` before committing.
- **JaCoCo floor** (`verify`): 20% line and branch per module bundle. A tripwire,
  not a target.

### Test conventions

- `*Test` = unit, no server, runs in `mvn test`. `*IT` = integration, needs the
  profile. Prefer unit tests; `postgresql-client`'s `MockServer` lets you drive real
  protocol sequences with no Docker, and most protocol behavior is testable that way.
- Tests run **serially** (`src/test/resources/junit-platform.properties`). Class-level
  parallelism is off on purpose: surefire and failsafe mis-file their XML under it,
  putting testcases in another class's report and printing per-class counts that are
  wrong (a class with 22 tests reported 0). Every test still ran, but a build that
  misreports itself is worse than a slower one; the file records the measurements and
  how to trade back. Integration tests share one container (`AbstractPostgresIT`), so a
  new `*IT` needs no special handling either way.
- Two opt-in switches are worth running a suite under when touching the areas they
  guard: `-Dorg.postgresql.client.checkRowLifetime=true` turns a read of a transient
  row after the cursor moved into an exception instead of silently wrong data
  (N7.3), and `-Pintegration-tests` is the Docker matrix. The row check is off by
  default because wrapping every row costs an allocation on the hot path.
- Any test that mutates process-global JVM state (`DriverManager`, default
  `Locale`/`TimeZone`, system properties) must be `@Isolated`. Better: design the
  test not to mutate global state.
- New functionality needs tests in the same series of commits. Performance work
  needs a test that pins the new behavior (see the recent `perf(core)` /
  `test(core)` commit pairs).

### Compat suites (expensive; do not run casually)

`compat-suites/pgjdbc` and `compat-suites/hibernate` run the upstream projects'
own suites against `postgresql-client-pgjdbc-compat` in Docker, and compare against
committed baselines in `<suite>/baselines/`. They take a long time and need
Docker plus network. Run one only when a change plausibly moves compat numbers,
and commit the refreshed baseline as its own `test(compat): refresh ... baseline`
commit. See each suite's `README.md`.

## Conventions

- **Plaintext ASCII only** in source, docs, comments, and commit messages.
- **Formatting:** whatever `spotless:apply` produces (4-space, braces on the same
  line). Match surrounding style; `.editorconfig` covers LF, final newline, and
  trailing whitespace.
- **Packages:** root is `org.postgresql.client`.
- **Naming:** clear and descriptive; public API should read naturally for
  PostgreSQL users and need not mirror JDBC terminology.
- **Nullability and immutability:** prefer immutable types and explicit optional
  handling. Make illegal states unrepresentable where practical.
- **Errors:** surface PostgreSQL errors with full fidelity (SQLSTATE, severity,
  message, detail, hint, position, schema/table/column, constraint). Never
  swallow protocol detail. See ADR-0007.
- **Secrets:** never logged; clearable (`char[]`/`byte[]`/suppliers) at our
  boundaries. See ADR-0010.
- **Server bytes are untrusted:** decoders must not crash or over-allocate on
  malformed input.
- **Comments:** explain why, not what. Match surrounding density.

## Working in this repo

- **One logical change per commit.** Commit messages are conventional-commit
  style with a module scope and, where applicable, the plan item in parentheses:
  `perf(core): binary parameter encoders for numeric and uuid (P4, N5.7)`,
  `fix(jdbc): bind SQLXML parameters with the xml type OID`.
  Scopes in use: `core`, `jdbc`, `protocol`, `compat`, `bench`, `docs`, `test`.
- Keep tests and plan/doc checkbox updates as **separate commits** from the code
  change they accompany.
- Never add `Co-Authored-By` or "Generated with" trailers.
- Make focused changes; do not add unrelated code, license headers, or CI
  scaffolding unless asked.
- When a change implements a plan item, tick its checkbox in
  `docs/plans/overall.md`; when it defers something, record it in
  `docs/follow-up.md` with enough detail (files, line hints, why) that a fresh
  agent can pick it up.
- Structural decisions (new dependency, module layout, package naming, async vs.
  blocking I/O, a public API shape) get raised explicitly, and land as an ADR if
  accepted.
- Update this file and the README when project-wide conventions or goals change.

## Definition of done

1. `./mvnw verify` passes (compile, unit tests, Spotless, ASCII, layering, coverage).
2. New behavior has a test; protocol behavior prefers `MockServer` over Docker.
3. If the change touches server-observable behavior, either
   `./mvnw verify -Pintegration-tests` passes locally or the gap is stated
   explicitly in the summary and, if it is real work, recorded in
   `docs/follow-up.md`.
4. Public packages added are exported in the module's `module-info.java`, and any
   new public type is listed in `docs/api-surface.md` (`ApiSurfaceManifestTest`
   fails the build otherwise).
5. Plan/doc updates committed separately.
6. Report honestly: if tests were skipped or a step was not run, say so.

## Common pitfalls

- Formatting failures only appear at `verify`; run `./mvnw spotless:apply` before
  committing. ASCII failures surface earlier, at `test`, so avoid pasting Unicode
  punctuation into source, docs, or commit messages.
- A new public package without a `module-info.java` export compiles in-module
  and breaks downstream.
- `synchronized` around blocking I/O pins virtual threads: use `ReentrantLock`.
  `NoSynchronizedSourcePolicyTest` rejects the keyword in shipped code, so this
  fails the build rather than showing up later as a latency mystery.
- Do not let pgjdbc's legacy behavior leak into core; it belongs in the JDBC and
  compat layers. Core changes for compat parity need a real core-level reason.
- `-Dtest=...` across the reactor fails modules that lack the class unless you
  pass `-Dsurefire.failIfNoSpecifiedTests=false`.
- Testcontainers work needs a running Docker daemon; if there is none, use
  `MockServer` or `-Dpg.it.host` instead of disabling tests.

## License

`pg-java` is released under the [PostgreSQL License](https://opensource.org/license/postgresql).
Contributions are accepted under the same license.

---
> Source: [pgjdbc/pg-java](https://github.com/pgjdbc/pg-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
