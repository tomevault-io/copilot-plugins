## openapi-request-validator

> **OpenAPI Request Validator** is a Java library for validating HTTP request/responses against OpenAPI/Swagger specifications. It's a production-ready, framework-agnostic library maintained by Atlassian with multiple adapter modules for popular testing and web frameworks.

# OpenAPI Request Validator - Agent Development Guide

## Project Overview

**OpenAPI Request Validator** is a Java library for validating HTTP request/responses against OpenAPI/Swagger specifications. It's a production-ready, framework-agnostic library maintained by Atlassian with multiple adapter modules for popular testing and web frameworks.

### Key Characteristics
- **Purpose**: Validate HTTP interactions against OpenAPI v3 and Swagger v2 specifications
- **Scope**: Standalone validation logic with adapters for various frameworks (Spring, Pact, WireMock, REST Assured, Ktor)
- **Build**: Maven 3.9+, JDK21+ required
- **License**: Apache 2.0

## Repository Structure

### Core Module
- **openapi-request-validator-core**: Core validation engine with framework-agnostic Request/Response abstractions
  - Located in `openapi-request-validator-core/src/main/java/com/atlassian/oai/validator/`
  - Key packages:
    - `interaction/`: Operation resolution, request/response validation
    - `model/`: Request/Response abstractions, Body types (String, ByteArray, InputStream)
    - `schema/`: JSON Schema validation, custom format validators
    - `report/`: Validation result reporting with configurable message levels
    - `whitelist/`: Error suppression rules for selective validation
    - `util/`: Helper utilities (content type, HTTP parsing, OpenAPI loading)

### Adapter Modules
- **openapi-request-validator-pact**: Pact consumer testing integration
- **openapi-request-validator-wiremock**: WireMock HTTP mocking validation
- **openapi-request-validator-wiremock-junit5**: WireMock JUnit5 integration
- **openapi-request-validator-restassured**: REST Assured testing framework integration
- **openapi-request-validator-mockmvc**: Spring MockMVC integration
- **openapi-request-validator-spring-webmvc**: Spring WebMVC runtime validation (production use)
- **openapi-request-validator-spring-web-client**: Spring WebClient HTTP client validation
- **openapi-request-validator-ktor-client**: Kotlin Ktor HTTP client validation

### Examples & Documentation
- **openapi-request-validator-examples**: Runnable examples for all adapters
- **docs/**: Feature documentation, OpenAPI v3 and Swagger v2 guides
  - **docs/adr/**: Architecture Decision Records documenting key design choices and constraints

## Architecture Decision Records

For comprehensive documentation of key architectural decisions, constraints, and design rationale, see [docs/adr/](docs/adr/):

- **ADR-001**: Framework-Agnostic Request/Response Abstraction - Core abstraction pattern enabling framework independence
- **ADR-002**: Plugin Architecture for Framework Adapters - Modular design for supporting multiple frameworks
- **ADR-003**: Streaming Body Handling Strategy - Performance and memory optimization for large payloads
- **ADR-004**: Schema Transformation Pipeline - Spec manipulation and OpenAPI compatibility
- **ADR-005**: OpenAPI Version Handling - Supporting v3.0, v3.1, and Swagger v2 specifications
- **ADR-006**: Validation Error Whitelisting - Flexible error suppression and selective validation
- **ADR-007**: Immutable Validation Reports - Thread-safe and reliable result reporting

When making architectural decisions, refer to these ADRs and follow the [TEMPLATE](docs/adr/TEMPLATE.md) for documenting new decisions.

## Development Guidelines

### Code Style & Quality
- Checkstyle enabled by default - follow existing code style
- Tests required for all new features and bug fixes
- Run full verification before PRs:
  ```
  mvn clean verify javadoc:javadoc
  ```

### Testing Strategy
- Comprehensive test coverage in `src/test/` directories
- Test resources (YAML/JSON specs) in `src/test/resources/oai/`
- When fixing bugs, focus on relevant test suites (don't need to fix unrelated failures)
- Use `mvn test` for quick validation

### Key Architectural Patterns

**Request/Response Abstraction**
- All validation works through `Request` and `Response` interfaces
- Multiple implementations: `SimpleRequest`, `SimpleResponse`, plus adapter-specific implementations
- Body handling supports streaming via `Body` interface with implementations for String, ByteArray, InputStream

**Validation Flow**
- `OpenApiInteractionValidator` is the main entry point
- `ApiOperationResolver` finds matching operations in the OpenAPI spec
- Operation-specific validators (`RequestValidator`, `ResponseValidator`) perform validation
- Results collected in `ValidationReport` with configurable message severity

**Configuration & Control**
- `ValidationConfiguration` for fine-grained validation control
- Custom validators: `CustomRequestValidator`, `CustomResponseValidator`
- Error whitelisting via `ValidationErrorsWhitelist` rules
- Schema transformers for spec manipulation (references, additional properties, etc.)

### Common Development Tasks

**Adding Validation Logic**
- Core validation lives in `openapi-request-validator-core/src/main/java/`
- Request validation: `interaction/request/` package
- Response validation: `interaction/response/` package
- Schema validation: `schema/` package
- Add corresponding tests in mirrored `src/test/` structure

**Adding Framework Support**
- Create new adapter module following naming pattern: `openapi-request-validator-{framework}`
- Implement `Request` and `Response` interfaces
- Follow existing adapter patterns (e.g., MockMVC, REST Assured)
- Add examples in `openapi-request-validator-examples`

**Updating OpenAPI Specs**
- Test specs in `src/test/resources/oai/v2/` and `oai/v3/` and `oai/v31/`
- Include both valid and invalid test cases
- Reference specs from test classes using `OpenApiLoader.loadOpenAPI()`

**Adding Message Customization**
- Validation message properties: `src/main/resources/swagger/validation/`
- `messages.properties`: Custom validation messages
- `default-levels.properties`: Message severity levels
- `LevelResolver` determines final message severity

### Build & Release Process

**Local Build**
```bash
mvn clean install                    # Quick build & test
mvn clean verify javadoc:jar        # Full verification
mvn test                            # Tests only
```

**Release (Maintainers Only)**
- Run: `./bin/prepare-release.sh [patch | minor]`
- Triggers `mvn release:prepare` with semver logic
- Release build published via CI/CD pipeline
- Update `RELEASE-NOTES.md` after release

### Semver Guidelines
- **Patch**: Bug fixes, minor improvements, patch dependency bumps
- **Minor**: New features, API additions, minor dependency bumps
- **Major**: Breaking changes, large refactors (requires discussion)

## Testing Patterns

### Test Organization
- Unit tests: Small, focused, no framework dependencies
- Integration tests: Full validation pipeline
- Example tests: Real-world usage patterns

### OpenAPI Specs for Testing
- `api-users.yaml/json`: Basic CRUD operations (users resource)
- `api-with-*.yaml`: Feature-specific specs (arrays, cookies, patterns, etc.)
- Invalid specs: `api-malformed.yaml/json`, specs with missing security schemes, etc.
- Composition: `api-composition.yaml`, `api-complex-composition.yaml`

### Request/Response Test Fixtures
- `requests/`: Raw HTTP request examples (formdata, multipart, JSON)
- `responses/`: JSON/XML response examples (valid and invalid)

## Important Notes

### Performance Considerations
- `ResettableInputStreamBody` used for request body caching in servlet context
- Benchmarks available: `RequestBodyValidationBenchmark.java`
- Stream-based body handling to minimize memory footprint

### Java Version Support
- **Core & Adapters**: JDK21+ required to build
- **Spring WebMVC**: Requires JDK17+, Spring Boot 4+, Jakarta namespace
- **Spring MockMVC & Web Client**: Spring 7+ compatible

### Framework Compatibility
- **WireMock**: Both JUnit 4 (legacy) and JUnit 5 modules provided
- **Spring**: Legacy branch support for older versions (critical fixes only)
- **Pact**: Integration with Pact-JVM consumer testing

## Contributing

See `CONTRIBUTING.md` for:
- Pull request guidelines
- CLA (Contributor License Agreement) requirements
- Release targeting (master vs. maintenance branches)
- Issue assignment process

---

**Last Updated**: 2026-02-18
**Maintainer Contact**: See CONTRIBUTING.md for Atlassian contact information

---
> Source: [atlassian/openapi-request-validator](https://github.com/atlassian/openapi-request-validator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
