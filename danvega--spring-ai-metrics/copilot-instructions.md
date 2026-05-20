## spring-ai-metrics

> - Build: `./mvnw clean install`

# AI Metrics Project - Developer Guide

## Build & Run Commands
- Build: `./mvnw clean install`
- Run: `./mvnw spring-boot:run`
- Test all: `./mvnw test`
- Run single test: `./mvnw test -Dtest=ClassName#methodName`
- Example: `./mvnw test -Dtest=AimetricsApplicationTests#contextLoads`

## Code Style Guidelines
- **Package Structure**: Use `dev.danvega.aimetrics` namespace
- **Naming**: 
  - Classes: PascalCase (e.g., `MetricsController`)
  - Methods/Variables: camelCase (e.g., `getMetrics()`)
- **Imports**: Standard ordering (Java → Spring → other libraries)
- **Testing**: Use JUnit 5 with Spring Boot Test framework
- **Error Handling**: Use Spring exception handlers for API errors
- **Documentation**: Add JavaDoc for public methods/classes

## Tech Stack
- Java 23
- Spring Boot 3.4.3
- Spring AI 1.0.0-M6
- Micrometer Prometheus (for metrics)

## Project Structure
Follow standard Spring Boot application architecture with controllers, services, and repositories as needed.

---
> Source: [danvega/spring-ai-metrics](https://github.com/danvega/spring-ai-metrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
