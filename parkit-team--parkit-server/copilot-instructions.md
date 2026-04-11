## parkit-server

> This repository is a small multi-service Kotlin/Spring Boot workspace.

# AGENTS.md (parkit-server)

This repository is a small multi-service Kotlin/Spring Boot workspace.
Each service is an independent Gradle project (there is no root Gradle build).

Repo layout
- `analysis-service/` (Spring WebFlux + Kafka + reactive Redis)
- `socket-service/` (Spring WebFlux + WebSocket + Kafka)
- `report-service/` (Spring MVC + Kafka + MongoDB)

Toolchain
- Kotlin: 2.2.21
- Spring Boot: 4.0.3
- Java toolchain: 17
- Gradle wrapper: 9.3.1

Cursor/Copilot rules
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` found in this repo at time of writing.

------------------------------------------------------------

Build / Run / Test (per service)

Important: `gradlew` is not executable in this repo.
Use `bash ./gradlew ...` (recommended) or make it executable once:

```bash
chmod +x analysis-service/gradlew report-service/gradlew socket-service/gradlew
```

From a service directory

```bash
# build (compiles + runs tests)
bash ./gradlew clean build

# run locally
bash ./gradlew bootRun

# run tests
bash ./gradlew test

# run a single test (same command works for all services)
bash ./gradlew test --tests "com.parkit.<service>.SomeTest"

# verification lifecycle (typically includes `test`)
bash ./gradlew check
```

Run a single test (JUnit 5)

Gradle supports selecting tests via `--tests`.

```bash
# single test class
bash ./gradlew test --tests "com.parkit.analysis.AnalysisServiceApplicationTests"

# single test method
bash ./gradlew test --tests "com.parkit.analysis.AnalysisServiceApplicationTests.contextLoads"

# wildcard match
bash ./gradlew test --tests "*ApplicationTests"

# multiple patterns (repeat --tests)
bash ./gradlew test --tests "*RiskDetectionServiceTest" --tests "*ParkingScoringServiceTest"
```

Notes
- If you run commands from the repo root, they will fail because there is no root build.
- If a test selection does not match, Gradle will report "No tests found"; verify the fully-qualified name.

Lint / Format
- No dedicated lint/formatter tasks (ktlint/detekt/spotless) are configured in `build.gradle.kts` today.
- Use `bash ./gradlew check` as the baseline verification step.
- If you introduce formatting changes, keep them minimal and consistent with existing files (see style section).

Local dependencies (typical)
- `analysis-service`: Kafka + Redis
- `socket-service`: Kafka
- `report-service`: Kafka + MongoDB
- Environment variables commonly used: `SPRING_KAFKA_BOOTSTRAP_SERVERS`, `SPRING_DATA_REDIS_HOST`, `SPRING_DATA_REDIS_PORT`, `SPRING_MONGODB_URI`

------------------------------------------------------------

Code Style (Kotlin + Spring)

General
- Prefer small, focused classes/functions; keep diffs minimal and consistent with existing conventions.
- Prefer `val` over `var`; avoid mutable shared state.
- Nullability is strict (`-Xjsr305=strict` is enabled); model nulls explicitly.

Formatting
- Follow existing formatting in the repo (current Kotlin sources use tabs for indentation).
- Use trailing commas where it improves diffs (especially in multiline parameter/argument lists).
- Keep line lengths reasonable; wrap long call chains.
- Prefer explicit units in names when values are ambiguous (e.g., `*Cm`, `*Deg`, `*Ms`).

Imports
- Avoid wildcard imports.
- Prefer explicit imports and keep them organized (IDE default is fine).
- Do not alias imports unless it removes a real ambiguity.

Packages / modules
- Keep code under the service package:
  - `analysis-service`: `com.parkit.analysis...`
  - `report-service`: `com.parkit.report...`
  - `socket-service`: `com.parkit.socket...`
- Mirror package structure to folders under `src/main/kotlin`.

Naming
- Classes / interfaces: `PascalCase`
- Functions / properties: `camelCase`
- Constants: `UPPER_SNAKE_CASE` (use `const val` when possible)
- Spring beans: use descriptive class names (`*Service`, `*Repository`, `*Controller`, `*Handler`)

Types and APIs
- Prefer immutable DTOs (`data class`) with explicit types.
- Prefer sealed hierarchies for domain outcomes when there are a few known variants.
- Avoid leaking framework types across layers unless it simplifies the design.
- Be careful with numeric conversions (Double -> Int rounding); document units at DTO boundaries.

Spring conventions
- Prefer constructor injection (no field injection).
- Keep configuration in `src/main/resources/application.yml` / `application.properties` (or introduce profiles deliberately).
- Separate concerns:
  - controllers/handlers: HTTP/WebSocket boundary, request/response mapping
  - services: business logic
  - repositories/clients: persistence and external I/O

Reactive vs blocking
- `analysis-service` and `socket-service` use WebFlux; avoid blocking calls on event-loop threads.
- `report-service` uses MVC (blocking). Keep DB/network calls inside request threads or dedicated executors.
- If you must bridge blocking work in WebFlux, isolate it explicitly (e.g., bounded elastic scheduler) and document why.

Error handling
- For HTTP APIs, prefer consistent error responses:
  - use `ResponseStatusException` for simple cases
  - use `@ControllerAdvice`/`@RestControllerAdvice` for shared mapping and payloads
- Don’t swallow exceptions; log with context and either recover explicitly or rethrow.
- Validate inputs at the boundary (request DTO validation) and fail fast.
- Kafka consumers: catch/ log with topic+key+offset when available; avoid poison-message infinite loops.

Logging
- Use structured, contextual logs (include identifiers like request id, entity id, topic/partition/offset).
- Avoid logging secrets (tokens, credentials, full payloads if they may contain PII).

Testing
- Test framework: JUnit 5 (Gradle uses `useJUnitPlatform()`).
- Prefer faster slice tests when possible:
  - MVC: `@WebMvcTest`
  - WebFlux: `@WebFluxTest`
  - Mongo: slice tests via repository tests or `@SpringBootTest` when needed
- Use `@SpringBootTest` only when you need the full context.
- Name tests clearly and keep assertions focused.

------------------------------------------------------------

Service-specific quick refs

`analysis-service/`
- Dependencies: WebFlux, Kafka, reactive Redis, coroutines-reactor.
- Test deps include: `spring-boot-starter-*-test` variants + coroutine test.
- Produces coaching events (Kafka `coaching-event`) and score results.

`socket-service/`
- Dependencies: WebFlux, WebSocket, Kafka, coroutines-reactor.
- Consumes coaching events and broadcasts over STOMP `/topic/coaching`.

`report-service/`
- Dependencies: WebMVC, Kafka, MongoDB.
- Persists driving sessions + sensor logs in MongoDB.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Parkit-Team)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Parkit-Team)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
