## bookk-me-backend

> Modular monolith split into Ktor microservices. Kotlin + Ktor (CIO) + Koin DI + Exposed ORM + kotlinx ProtoBuf serialization + MockK/JUnit5 tests. Follow the recipes below verbatim; they mirror the real code (reference implementation: `service/appointments`, the newest service).

# bookk-server — Agent Playbook

Modular monolith split into Ktor microservices. Kotlin + Ktor (CIO) + Koin DI + Exposed ORM + kotlinx ProtoBuf serialization + MockK/JUnit5 tests. Follow the recipes below verbatim; they mirror the real code (reference implementation: `service/appointments`, the newest service).

## Module map

```
core/                    shared infra: domain Result/Error types, service (Ktor server, auth, respondWith), data (Exposed, event streaming, cache)
library/                 money, idempotency, permissions
service/<svc>/
  domain/api/            operation interfaces + entities + <Svc>ErrorCodes   (package com.bookk.<svc>.domain.api.{operation,entity})
  domain/impl/           <Operation>Impl + di/DI.kt + tests                  (package com.bookk.<svc>.domain.impl.{operation,di})
  data/source/           datasource INTERFACES                               (package com.bookk.<svc>.domain.datasource)
  data/                  datasource impls, orm tables/entities, migration, di
  microservice/          route/<Svc>Routing.kt (typed Resources), route/api/*Route.kt, <Svc>Microservice.kt main, route tests
  client/                cross-service events/API
```

New gradle modules must be registered in `settings.gradle.kts` (one `include` per submodule, grouped per service). Convention plugins (`libs.plugins.bookk.microservice`, `bookk.domain.impl`, `bookk.domain.api`, `bookk.data`, …) already add Ktor/Koin/MockK/kotlin-test deps and JUnit platform — do not re-add them.

**Never generate database migrations.** When an ORM table changes (new column, new index, etc.), leave the table definition updated but do NOT bump `referenceVersion`/`targetVersion` in `<Svc>Migration.kt` or run its `main()` to produce a new `V<n>__migration_script.sql`. The user runs that generation step themselves.
**Ignore all git operations.** Managing Git is the responsibility of the developer, do not automatically commit, push or merge

## Core conventions (apply everywhere)

- **Never write comments.** No `//` comments, no KDoc, no explanatory blocks — in production code or tests. Express intent through names and structure instead: rename the function, extract the condition into a named value, split the method. If something seems to need a comment, that is a signal the code should be clearer. The only exceptions are the route KDoc the openApi plugin parses (`Summary:`, `Body:`, `Response:`, …) and comments that already exist in the file — never delete or reword someone else's comment.
- Operations return `Result<T>`; impls **throw** errors inside `transactionManager.transaction { }`, which catches and converts to `Result.failure`.
- Domain errors: nested `sealed interface Error` in the operation interface; each case is a `class` extending `BusinessError(statusCode, code, message)` (classes, NOT data objects). Assert with `is`, never equality.
- Error codes live in `domain/api/.../<Svc>ErrorCodes` as `BASE + n`. Blocks: auth=0, user=100000, business=200000, appointments=300000. Next service takes the next 100000 block.
- Generic infrastructure errors: `com.bookk.core.domain.entity.Error` (`NotFound`, `OperationNotAllowed`, …).
- `call.respondWith(result)` (core/service) maps: success Unit→204, success T→200, `BusinessError`→its statusCode + `SimpleServerError(errorCode, message)`, `Error.NotFound`/`Error.OperationNotAllowed`→**404** (intentional: permission failures do NOT return 403), anything else→500 (logged).
- Permissions: `permissionsDataSource.getPermissions(userId, businessId).assert(ObjectPermission.EDIT)` (library/permissions) — throws `Error.OperationNotAllowed`.
- Wire format is ProtoBuf (`application/x-protobuf`) for all bodies/responses. **A nullable collection (`List<T>?`, `Map<K, V>?`) cannot be serialized when null** — kotlinx throws `'null' is not supported as the value of collection types in ProtoBuf`. For an optional group of fields in a partial-update DTO, wrap them in a nullable `@Serializable` holder class (a nullable message is fine) instead of making each list nullable — see `BusinessUpdateModel.schedule: Schedule?`. **A nullable property must not also have a default** — the serializer runs with `encodeDefaults = true`, and encoding a defaulted null throws `'null' is not supported for optional properties in ProtoBuf` as soon as a caller omits it. Give every nullable field on a partial-update DTO no default at all and pass them explicitly (`BusinessUpdateModel`, `UserEditModel`).
- Entities: `@Serializable data class` in `domain/api/.../entity` with a `companion object { fun stub(...) }` factory (defaulted params, `Uuid.random()`, `Instant.fromEpochMilliseconds(0)`) — add `stub()` to every new entity; tests rely on it.

## Recipe: new business operation

Files to touch (example names from appointments):

1. `domain/api/.../operation/DoThing.kt` — interface + errors:
```kotlin
interface DoThing {
    suspend operator fun invoke(userId: Uuid, ...): Result<Thing>

    sealed interface Error {
        class ThingExists : BusinessError(
            statusCode = HttpStatusCode.UnprocessableEntity.value,
            code = SvcErrorCodes.THING_EXISTS,
            message = "Thing already exists"
        ), Error
    }
}
```
2. Add the code constant to `<Svc>ErrorCodes`.
3. `domain/impl/.../operation/DoThingImpl.kt`:
```kotlin
internal class DoThingImpl(
    private val thingDataSource: ThingDataSource,
    private val permissionsDataSource: PermissionsDataSource,
    private val transactionManager: TransactionManager,
    private val eventProducer: StandardEventProducer, // only if events sent
) : DoThing {
    override suspend fun invoke(userId: Uuid, ...): Result<Thing> = transactionManager.transaction {
        val thing = thingDataSource.get(id) ?: throw Error.NotFound()
        permissionsDataSource.getPermissions(userId, businessId).assert(ObjectPermission.EDIT)
        if (conflict) throw DoThing.Error.ThingExists()
        thingDataSource.create(...) // also { eventProducer.send(SvcEvent.X(...)) } if needed
    }
}
```
4. Register in `domain/impl/.../di/DI.kt`: `factoryOf(::DoThingImpl) bind DoThing::class`.
5. If a new datasource method is needed: add to the interface in `data/source/.../datasource/`, implement in `data/.../datasource/<X>DataSourceImpl.kt` (`internal class ... : DataSource(), XDataSource`, queries wrapped in `dbQuery { }`, Exposed v1 DSL, `Uuid` for ids). New datasources are registered in `data/.../di/`: `singleOf(::XDataSourceImpl) bind XDataSource::class`. New tables go in `data/.../orm/{table,entity}` and must be added to the service's `<Svc>Migration.kt` `tables()` array.

### Writes live on the entity, not in the datasource

Mapping a domain model onto rows belongs in the DAO entity's companion, so a datasource method stays a one-liner and the field list has exactly one home. Reference: `AppointmentBusinessEntity`, `BusinessEntity`, `AppointmentEntity`.

```kotlin
internal class ThingEntity(id: EntityID<Uuid>) : UuidEntity(id) {
    var name by ThingTable.name

    fun domain(): Thing = Thing(id = id.value, name = name)

    private fun replaceChildren(children: List<Child>) {
        val entityId = id.value // `id` inside a table lambda resolves to the table's column, not the entity
        ChildTable.deleteWhere { ChildTable.thingId eq entityId }
        ChildTable.batchInsert(children) { this[ChildTable.thingId] = entityId /* … */ }
    }

    companion object : DecoratorUUIDEntityClass<ThingEntity>(ThingTable) {
        fun new(model: Thing): ThingEntity = new { name = model.name }.apply { replaceChildren(model.children) }

        fun findByIdAndUpdate(model: ThingUpdate) = findByIdAndUpdate(model.id) {
            model.name?.let { name -> it.name = name }
            it.updatedAt = Clock.System.now()
        }
    }
}
```
```kotlin
override suspend fun create(model: Thing): Thing = dbQuery { ThingEntity.new(model).domain() }
override suspend fun update(model: ThingUpdate): Thing = dbQuery {
    (ThingEntity.findByIdAndUpdate(model) ?: throw Error.NotFound()).domain()
}
```

Rules that fall out of this:
- **Never call `flush()` or `refresh()`.** Exposed flushes its entity cache before executing any statement, so a staged insert/update is always written before the next query or DSL statement observes it. Every such call added during this codebase's history turned out to be dead — `refresh()` is worse than dead, it re-reads the row and discards pending writes, which then needs a `flush()` before it to compensate.
- `findByIdAndUpdate` (Exposed's own) reads with `SELECT … FOR UPDATE`. When an update is conditional (an event-ordering guard, a state check), put the condition **inside** that block so the check and the write share the row lock. Do not hand-roll `findById` + check + mutate; that drops the lock and reopens the race.
- Return `Unit` from datasource writes unless a caller branches on the result. A `Boolean` that only tests assert on is dead API; assert the guard through the stored state instead.
- Read a row back with `findById(...)?.toDomain()`, not a hand-rolled `selectAll().where { }` + `wrapRow`. Do not re-read after a write either — `new(...)` and `findByIdAndUpdate(...)` return the entity, so `.toDomain()` on it is enough.
- Entity referrers lazy-load one query per parent, so a read returning **many** rows needs `.with(Entity::children)` to batch them. Today's reads are all effectively single-row (`BusinessTable.userId` is a `uniqueIndex`, so a user owns at most one business), so plain referrers are correct — revisit if that changes.
6. Write the impl unit test (see Testing) — every `Error` case + success + event publication if any.

## Recipe: new Ktor route

1. Add a typed resource to `route/<Svc>Routing.kt` (nested `@Resource` classes with `val parent: X = X()` chain):
```kotlin
@Resource("/{id}/cancel")
class Cancel(val parent: Appointment = Appointment(), val id: Uuid)
```
2. Add/extend a `fun Routing.xyz()` in `route/api/<Thing>Route.kt`. Protected routes go inside `authenticate { }`; inject operations lazily inside the handler; request-only DTOs are `@Serializable internal class` at the top of the route file:
```kotlin
fun Routing.thing() {
    authenticate {
        /**
         * Summary: Create thing
         * Description: Create new thing from request
         * Tag: thing
         * Security: jwt
         * Body: application/x-protobuf [com.bookk.svc.microservice.route.api.ThingRequest]
         * Response: 200 application/x-protobuf [com.bookk.svc.domain.api.entity.Thing] Created thing
         * Response: 422 application/x-protobuf [com.bookk.core.domain.entity.SimpleServerError] Create thing errors<br>THING_EXISTS (300010) Thing already exists
         */
        post<Api.Thing> {
            val principal = requireNotNull(call.principal<AppPrincipal>())
            val body = call.receive<ThingRequest>()
            val doThing by application.inject<DoThing>()

            call.respondWith(doThing(userId = principal.userId, ...))
        }
    }
}
```
3. KDoc OpenAPI rules (the ktor openApi plugin parses these): `Summary:`, optional `Description:`, `Tag:`, `Security: jwt` only when inside `authenticate {}`, **`Body:` (NEVER `RequestBody:`)**, one `Response:` line per status. Fully qualified type names in brackets. Every case of the operation's `sealed interface Error` must appear under the 422 response with `NAME (<n>) message`. 204 responses: `Response: 204 application/x-protobuf <description>` (no type). **Never use a colon (`:`) anywhere in the KDoc text after the field prefix** (e.g. `NAME (<n>) message`, not `NAME (<n>): message`) — the openApi plugin parses on `:` and an extra one breaks the field. **NEVER put a generic type in a `Response:` KDoc line** (e.g. `[kotlin.collections.List<Foo>]` silently produces no schema — the compiler plugin's `TypeReference$Link$Reference.asIrType()` passes the raw string including `<…>` to `FqName()` then `ClassId.topLevel()`, which finds no class and returns null). For any `List<T>` response, omit the `Response:` line from KDoc and chain a `.describe {}` block instead:
```kotlin
        /**
         * Summary: Get things
         * Tag: thing
         * Security: jwt
         */
        get<Api.Things> {
            // ...
        }.describe {
            responses {
                response(HttpStatusCode.OK.value) {
                    schema = jsonSchema<List<Thing>>()
                    description = "List of things"
                    ContentType.Application.ProtoBuf()
                }
            }
        }
```
Required imports: `io.ktor.http.ContentType`, `io.ktor.openapi.jsonSchema`, `io.ktor.server.routing.openapi.describe`.
4. When a path id duplicates a body id, validate: `if (it.id != body.id) call.respond(HttpStatusCode.BadRequest, "Invalid request") else ...`.
5. Register the new route fn in `route/<Svc>Route.kt` aggregator (`fun Routing.<svc>Route()`); the aggregator is already wired in `<Svc>Microservice.kt`.
6. `AppPrincipal` fields: `authId`, `userId`, `deviceId` (all `Uuid`).

## Testing

Run: `./gradlew :service:<svc>:microservice:test` / `:service:<svc>:domain:impl:test` (append `--tests "com.bookk...ClassName"` to filter).

Hard rules (enforced by fixtures or review):
- NEVER `runBlocking`. Operation tests: `runUnitTest { }`; route tests: `routeTest { }` (both from core fixtures).
- `given()` / `whenn()` / `then()` markers are **required in every test** — `runUnitTest` asserts at runtime that all three were called; a test without them fails.
- Fresh SUT and fresh mocks per test — no sharing across tests, no class-level mocks.
- DO NOT EDIT `core/src/testFixtures/kotlin/com/bookk/core/test/Test.kt`.
- Test names: backticked sentences, e.g. `` fun `should return failure when request overlaps with existing appointment`() ``.
- Use entity `stub()` factories instead of hand-built instances; pass only the fields the test depends on. Provide real entity instances to mocks (ProtoBuf serialization NPEs otherwise).
- JUnit assertions (`org.junit.jupiter.api.Assertions`). Assert error types with `is`: `assertTrue(result.exceptionOrNull() is DoThing.Error.ThingExists)`.
- If the operation sends events: include a test with `coVerify(exactly = 1) { eventProducer.send(any(SvcEvent.X::class), any()) }`.

### Operation (domain/impl) test template

Using testing rules from AGENTS.md cover all ViewModels, Use cases(operations) and Data sources with unit tests. Afrer covering each feature (appointments, business etc.) Do a code review of your work and fix the issues if encountered. Make sure tests are fast, repeatable. Create utility functions or testFixtures in the core module for tools that are used across each tests. Do not share the state between unit tests, use private class SutFixture {
val thingArgParameter = mockk<SomeArgClass>()
val sut = ClassUnderTest(thingArgParameter)
} pattern to init SUT to make sure new instance created for every unit test. Use mockk kotlinx.test and junit5 primarily to write unit test. After the work is done if any new useful knowledge acquired, insert it in AGENTS.md file.

```kotlin
internal class DoThingImplTest {

    private class SutFixture {
        val thingDataSource = mockk<ThingDataSource>()
        val permissionsDataSource = mockk<PermissionsDataSource>()
        val transactionManager = mockk<TransactionManager>()
        val sut = DoThingImpl(thingDataSource, permissionsDataSource, transactionManager)
    }

    @Test
    fun `should create thing successfully`() = runUnitTest {
        given()
        val userId = Uuid.random()
        val thing = Thing.stub(userId = userId)
        val fixture = SutFixture()
        with(fixture) {
            coEvery { permissionsDataSource.getPermissions(userId, thing.businessId) } returns ObjectPermission.EDIT.int
            coEvery { thingDataSource.create(any()) } returns thing
            transactionManager.mockTransaction() // testFixtures(projects.core.domain.datasource)
        }

        whenn()
        val result = fixture.sut.invoke(userId, thing)

        then()
        assertTrue(result.isSuccess)
        assertEquals(thing, result.getOrNull())
    }
}
```
Permission-denied case: stub `getPermissions` to return `ObjectPermission.READ.int`, assert `result.exceptionOrNull() is Error.OperationNotAllowed`.

### Route (microservice) test template

```kotlin
internal class CreateThingTest {

    @Test
    fun `should create thing successfully`() = routeTest {
        given()
        val useCase: DoThing = mockk()
        val userId = Uuid.random()
        val thing = Thing.stub(userId = userId)
        coEvery { useCase.invoke(userId, any()) } returns Result.success(thing)

        setupApplication(
            extension = {
                install(Authentication) {
                    provider {
                        authenticate { context ->
                            context.principal(AppPrincipal(Uuid.random(), userId, Uuid.random()))
                        }
                    }
                }
            },
            diModule = module { single { useCase } },
            routeUnderTest = { thing() }
        )

        whenn()
        val client = createTestClient()
        val response = client.post(SvcRouting.Api.Thing()) { setBody(ThingRequest(...)) }

        then()
        assertEquals(HttpStatusCode.OK, response.status)
    }
}
```
- Domain-error case: `coEvery { ... } returns Result.failure(DoThing.Error.ThingExists())`, then assert status 422 and `response.body<SimpleServerError>().errorCode == SvcErrorCodes.THING_EXISTS`.
- Unauthorized case: `extension = { install(Authentication) { bearer { authenticate { null } } } }`, assert 401. Omit `Security` mocking nothing else.
- Permission-denied surfaces as **404** through `respondWith` — assert 404, not 403.
- Requests use typed resources (`client.post(SvcRouting.Api.Thing())`), never string paths. `createTestClient()` already sets ProtoBuf content type.
- Fixtures come from: `testImplementation(testFixtures(projects.core))` (given/whenn/then, runUnitTest), `testFixtures(projects.core.service)` (routeTest, setupApplication, createTestClient), `testFixtures(projects.core.domain.datasource)` (mockTransaction). Add `libs.joda.money` if entities use Money. Check the module's `build.gradle.kts` before adding — appointments modules already have these.

### Mandatory coverage (contract-first)

Before writing code, enumerate and track as a checklist:
1. Happy path.
2. EVERY case of the operation's `sealed interface Error` — at BOTH levels: impl test (exception type) and route test (HTTP status + `SimpleServerError.errorCode`).
3. Auth: 401 unauthenticated; permission-denied (impl: `Error.OperationNotAllowed`; route: 404).
4. Event publication, if the operation sends events.

Finish by emitting a Final Coverage Report mapping every checklist item to its test:
```
### Final Coverage Report
- [x] ThingExists: impl exception test + route 422/THING_EXISTS test
- [x] Unauthorized: route 401 test
```
Report any gaps explicitly — never silently skip a case.

---

### Development Workflow: Test-Driven Development

Always implement features using strict TDD. For every requested feature or change:

1. **Red** — Write a failing test first that captures the expected behavior. Do not write any implementation code before the test exists.
2. **Green** — Write the minimum implementation needed to make the test pass.
3. **Refactor** — Clean up the implementation and tests while keeping all tests green.

Rules:
- Never write production code without a failing test that requires it.
- Run the test suite after each step and confirm the expected pass/fail state before proceeding.
- Keep tests small and focused; one behavior per test.
- If a requirement is ambiguous, write the test that encodes your assumption and state it explicitly.

---

## Datasource (H2 integration) test conventions

`createTestDatabase(vararg tables: Table)` in `core/data/src/testFixtures` creates a per-test H2 database in `MODE=MySQL`. Call datasource methods inside `suspendTransaction { fixture.sut.method() }` within `runUnitTest { }`. Use `dbQuery`-based methods inside `suspendTransaction {}`. Cache-based methods (those using `mapExceptions` without `dbQuery`) are called directly without wrapping.

### CacheClient in datasource tests

Use `InMemoryCacheClient` from `testFixtures(projects.core.data.cache.impl)` — do NOT use `mockk<CacheClient<String>>`. The relaxed mock returns a non-null mock for `get()` which fails the reified cast. Example fixture:
```kotlin
private class SutFixture {
    val db = createTestDatabase(...)
    val cacheClient = InMemoryCacheClient()
    val sut = MyDataSourceImpl(cacheClient)
}
```
See `PassKeyDataSourceImplTest` and `AppointmentRequestDataSourceImplTest` as reference.

### H2 dialect limitations — untestable operations

These throw `UnsupportedByDialectException` in H2 even with `MODE=MySQL`. Omit their tests and leave a comment in the test file:

| Operation | Used by | Workaround for reads |
|---|---|---|
| `updateReturning` | — | rewrite as `update {}` + a follow-up `selectAll()` read, which H2 supports and makes the method testable (`BusinessDataSourceImpl.updateBusiness` was converted this way) |
| `deleteReturning` | `BusinessDataSourceImpl.deleteUserBusinesses`, `DeviceDataSourceImpl.deleteInactiveDevices` | — |
| `upsertReturning` | `NotificationSettingsDataSourceImpl.upsert(settings)` | insert via DAO entity directly |
| `upsert(where = …)` | — | split into `insert`/`update` datasource methods and let the operation choose (`NotificationTargetDataSourceImpl` + `UpdateTargetInformation`) |

For read methods whose writes use an unsupported upsert, insert test data directly via the DAO entity (e.g. `NotificationSettingsEntity.new(id) { ... }`).

Plain `upsert {}` (no `where` parameter) **is** supported by H2 and resolves conflicts via `uniqueIndex` columns — use it freely in datasource tests. Passing `where` to it throws `UnsupportedByDialectException`. Split a conditional upsert into separate `insert`/`update` datasource methods — the guard lives in the `update {}` where clause and returns whether a row matched — and let the operation decide, inside its `transactionManager.transaction { }`, whether to insert.

### Asserting unique-constraint violations

`runUnitTest` cannot be nested inside `assertThrows` (its body is `suspend`, and the outer `TestHolder` context would be lost). Assert constraint failures with `runCatching` instead:
```kotlin
whenn()
val result = runCatching { suspendTransaction { fixture.sut.createX(duplicate) } }

then()
assertTrue(result.exceptionOrNull() is Error.UniqueConstraintFailed)
```
`dbQuery { }` maps the driver exception to `com.bookk.core.domain.entity.Error.UniqueConstraintFailed`, which operations turn into a domain error via `.onConstraintFailure { }`.

### Batch-update / batch-cancel patterns

`markCompleted(before: Instant)` and `cancelOutdated(before: Instant)` update rows where `dateEnd < before`. Test the happy path by creating entities at `Instant.fromEpochMilliseconds(0)` (epoch → well in the past) and calling with `Clock.System.now()`. Test the boundary by creating entities at `Clock.System.now() + 24.hours` and verifying they are NOT affected.

### Cleanup methods (deleteDayOffsInThePast)

Insert `DayOffRange(LocalDate(2020,1,1), LocalDate(2020,1,2))` for a past day-off and `DayOffRange(LocalDate(2099,12,30), LocalDate(2099,12,31))` for a future one. After calling `deleteDayOffsInThePast()` re-read via `sut.get(businessId)` and assert counts.

---
> Source: [an22/bookk.me-backend](https://github.com/an22/bookk.me-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
