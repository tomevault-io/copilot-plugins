## nx-kafka

> Lightweight Kafka client library for Java 8+ game server cores.

# nx-kafka

Lightweight Kafka client library for Java 8+ game server cores.

## Project

- **Organization:** nexuslabsio
- **Package:** `io.nexuslabs.kafka`
- **Java:** 8 (source + target), tests run on Java 11+
- **Build:** Gradle (Kotlin DSL)
- **Dependencies:** kafka-clients:3.6.2, gson:2.13.2, slf4j-api:2.0.17 (compileOnly)

## Testing

- **Naming**: `methodName_shouldExpectedBehavior` or `methodName_shouldExpectedBehavior_whenCondition`
- Integration tests require Docker and are auto-skipped without it (`@Testcontainers(disabledWithoutDocker = true)`)

## Rules

- README.md is public documentation — keep it in sync with any API or behavior changes
- Never throw exceptions that could propagate to game server threads uncaught
- All code must compile to Java 8 bytecode
- SLF4J is optional (compileOnly) — always use internal NxLog facade, never import SLF4J directly in library code
- Compression libs (snappy, lz4, zstd) must stay excluded from transitive dependencies
- Dependency versions are managed in `gradle/libs.versions.toml` — never hardcode versions in `build.gradle`
- Public interfaces and API contracts used directly by library consumers must have Javadoc
- Do not add Co-Authored-By to commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nexuslabsio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
