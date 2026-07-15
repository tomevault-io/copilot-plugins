## lexiconmeum

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project Snapshot

LexiconMeum is a Java 21 Spring Boot backend for a Latin vocabulary search and grammar tool. It loads pre-parsed Wiktionary/Kaikki lexical data, builds in-memory lookup structures, and exposes REST endpoints for autocomplete and lexeme detail views.

## Canonical Commands

- Run all tests: `./mvnw test`
- Run one test class: `./mvnw -Dtest=ClassName test`
- Run one test method: `./mvnw -Dtest=ClassName#methodName test`
- Package the app: `./mvnw clean package`
- Run locally: `./mvnw spring-boot:run -Dspring-boot.run.profiles=local`

Prefer the Maven wrapper over a system Maven install.

## Package Map

- Wiktionary parsing/staging/linking: `src/main/java/com/annepolis/lexiconmeum/ingest/wiktionary`
- Raw tag mapping: `src/main/java/com/annepolis/lexiconmeum/ingest/tagmapping`
- Core domain model: `src/main/java/com/annepolis/lexiconmeum/shared/model`
- Shared reader/sink abstractions and exceptions: `src/main/java/com/annepolis/lexiconmeum/shared`
- In-memory lexeme cache: `src/main/java/com/annepolis/lexiconmeum/cache/inmemory`
- API routes, response keys, CORS, properties: `src/main/java/com/annepolis/lexiconmeum/webapi`
- Autocomplete/text search: `src/main/java/com/annepolis/lexiconmeum/webapi/bff/textsearch`
- Lexeme detail response assembly: `src/main/java/com/annepolis/lexiconmeum/webapi/bff/lexemedetail`
- Runtime config and bundled lexical data: `src/main/resources`
- Tests: `src/test/java`
- Test fixtures and test config: `src/test/resources`

## Architecture Pointers

- Architecture and module responsibilities: `ARCHITECTURE.md`
- Public context, endpoint examples, deployment notes: `README.md`
- Planned direction: `ROADMAP.md`

Treat source and tests as the authority for current behavior.

## Testing Expectations

Add focused tests when behavior changes.

- Parser, tag mapping, model: unit tests near the affected package
- Web API behavior: controller or integration coverage
- Narrow refactor: closest existing tests, plus `./mvnw test` when practical

Do not add tests for documentation-only changes unless explicitly asked.

## Data And Fixtures

The JSONL files under `src/main/resources` and `src/test/resources` are lexical data inputs and fixtures. Keep changes to them intentional and small. When changing parser behavior, prefer adding or editing the smallest relevant test fixture instead of broad data churn.

`src/main/resources/lexicalDataPartial.jsonl` is bundled application data. Avoid reformatting or mass-editing it as part of unrelated changes.

## API Conventions

Route constants live under `webapi`, especially `ApiRoutes`. Keep endpoint paths and response-key constants centralized instead of duplicating literals through controllers or tests.

The BFF packages should expose response shapes useful to the frontend rather than leaking parser internals directly.

## Implementation Preferences

- Follow existing Spring configuration patterns before introducing new infrastructure.
- Prefer domain objects and typed grammar enums over raw strings for grammatical concepts.
- Keep parsing concerns in `ingest` and API presentation concerns in `webapi/bff`.
- Keep shared abstractions small; avoid moving feature-specific behavior into `shared` unless it is genuinely reused.
- Preserve existing release/deploy conventions unless the task is specifically about changing them.

## Local Noise

Ignore IDE files, OS metadata, logs, dependency folders, and Maven build output. Do not treat local `.DS_Store`, `logs/`, `node_modules/`, or `target/` changes as meaningful project changes.

---
> Source: [annedyne/lexiconmeum](https://github.com/annedyne/lexiconmeum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
