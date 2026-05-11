## rabbitmq-client-java

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**RabbitMQ Client Library (Java)** is a production-ready client library for RabbitMQ with integrated HashiCorp Vault support for dynamic credential management. Built with Spring Boot 3.2 and Java 21.

## Technology Stack

- **Java**: 21 (LTS with virtual threads)
- **Build Tool**: Maven
- **Framework**: Spring Boot 3.2, Spring Cloud 2023
- **RabbitMQ Client**: Spring AMQP
- **Vault Integration**: Spring Cloud Vault
- **Circuit Breaker**: Resilience4j
- **Metrics**: Micrometer + Prometheus
- **Logging**: SLF4J + Logback
- **Testing**: JUnit 5, Mockito, Testcontainers

## Repository Structure

```
rabbitmq-client-java/
├── pom.xml                         # Parent POM
├── CLAUDE.md                       # This file
├── README.md                       # Documentation
├── Jenkinsfile                     # Jenkins CI/CD
├── .github/workflows/java-ci.yml   # GitHub Actions
├── rabbitmq-client/                # Core library module
│   ├── pom.xml
│   └── src/main/java/com/shoppingcart/rabbitmq/
│       ├── config/                 # Configuration classes
│       ├── connection/             # Connection management
│       ├── vault/                  # Vault integration
│       ├── publisher/              # Message publishing
│       ├── consumer/               # Message consumption
│       ├── retry/                  # Retry logic
│       ├── circuitbreaker/         # Circuit breaker
│       ├── metrics/                # Prometheus metrics
│       ├── health/                 # Health indicators
│       ├── dlq/                    # Dead letter queue
│       └── exception/              # Custom exceptions
├── rabbitmq-cli/                   # CLI tools module
│   ├── pom.xml
│   └── src/main/java/com/shoppingcart/rabbitmq/cli/
├── rabbitmq-examples/              # Examples module
│   ├── pom.xml
│   └── src/main/java/com/shoppingcart/rabbitmq/examples/
└── docs/                           # Documentation
```

## Quick Start

```bash
# Run the demo (recommended - single command, no setup needed)
make demo

# This will:
# 1. Build the project
# 2. Set up exchange and queue
# 3. Start a consumer
# 4. Publish 8 test messages
# 5. Display received messages
# 6. Clean shutdown
```

**Prerequisites**:
- Java 21 (`brew install openjdk@21`)
- RabbitMQ running on `localhost:30672` (or configure via env vars)
- Set `JAVA_HOME=/path/to/openjdk@21`

## Build Commands

```bash
# Using Makefile (recommended)
make build        # Compile all modules
make test         # Run unit tests
make package      # Build JAR packages
make install      # Install to local Maven repo
make demo         # Run combined publisher+consumer demo

# Using Maven directly
mvn clean install
mvn clean install -DskipTests
mvn test
mvn verify -P integration-tests
mvn jacoco:report
```

## Key Design Principles

1. **Vault Integration First**: All credentials from HashiCorp Vault
2. **Automatic Reconnection**: Exponential backoff with jitter
3. **Circuit Breaker**: Resilience4j for fault tolerance
4. **Observability**: Micrometer metrics, Spring Actuator health
5. **Spring Boot Native**: Auto-configuration, @ConfigurationProperties

## Configuration

Configuration via `application.yml` or environment variables:

```yaml
rabbitmq:
  host: localhost
  port: 5672
  vhost: /
  use-tls: false

  pool:
    size: 10
    prefetch-count: 10

  retry:
    max-attempts: 3
    initial-interval: 1s
    max-interval: 30s
    multiplier: 2.0

  circuit-breaker:
    enabled: true
    failure-threshold: 5
    success-threshold: 2
    timeout: 30s

spring:
  cloud:
    vault:
      uri: http://localhost:8200
      token: ${VAULT_TOKEN}
      rabbitmq:
        role: rabbitmq-role
        backend: rabbitmq
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `VAULT_ADDR` | Vault server address | `http://localhost:8200` |
| `VAULT_TOKEN` | Vault authentication token | - |
| `RABBITMQ_HOST` | RabbitMQ hostname | `localhost` |
| `RABBITMQ_PORT` | RabbitMQ port | `5672` |

## Testing

### Unit Tests
```bash
mvn test
```

### Integration Tests (requires Docker)
```bash
mvn verify -P integration-tests
```

### Coverage Report
```bash
mvn jacoco:report
open rabbitmq-client/target/site/jacoco/index.html
```

## Code Style

- Follow Google Java Style Guide
- Use `@Slf4j` for logging
- Prefer records for DTOs
- Use `Optional` instead of null
- Use `var` for local variables when type is obvious

## Commit Message Format

```
type(scope): subject

body

footer
```

Types: `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `chore`

## Current Development Status

### What Works
- **`make demo`** - Combined publisher+consumer demo (recommended)
- **Core library** - Publisher, Consumer, Connection pooling, Retry, Circuit breaker
- **Static credentials** - Works when `rabbitmq.vault.enabled=false`
- **Vault credentials** - Works when Vault is configured

### Known Issues
- **CLI tools (`make cli-demo`)** - Broken due to Spring Boot component scanning
  - Both `PublisherCli` and `ConsumerCli` are in the same JAR
  - Both implement `CommandLineRunner`, so Spring runs both with same args
  - **Workaround**: Use `make demo` instead

### Demo Configuration
The demo profile (`application-demo.yml`) uses:
- Port: `30672` (Kubernetes NodePort)
- Credentials: `demo/demo`
- Vault: disabled
- Circuit breaker: disabled

To use different settings:
```bash
RABBITMQ_HOST=localhost RABBITMQ_PORT=5672 RABBITMQ_USERNAME=guest RABBITMQ_PASSWORD=guest make demo
```

## Related Repositories

- **rabbitmq-client-go**: Go implementation (complete)
- **rabbitmq-client-library**: Python implementation (complete)
- **shopping-cart-infra**: RabbitMQ infrastructure (K8s, Vault)

---
> Source: [wilddog64/rabbitmq-client-java](https://github.com/wilddog64/rabbitmq-client-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
