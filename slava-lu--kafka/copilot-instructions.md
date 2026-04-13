## kafka

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Spring Boot 3.5.7 + Kafka microservice demonstrating producer-consumer patterns with PostgreSQL persistence. Java 21 required.

## Development Guidelines
Before writing any code, always propose a clear solution plan.
The plan must include:
- What approach you will take
- Key steps or architecture
- Possible alternatives (if relevant)
- Only after approval, proceed with implementing the code.

## Build & Run Commands

```bash
# Build
./mvnw clean install          # Clean build with tests
./mvnw compile                # Compile only (also generates OpenAPI code)
./mvnw package                # Create JAR

# Run
./mvnw spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=local"

# Test
./mvnw test                   # Run all tests
./mvnw test -Dtest=ClassName  # Run single test class

# Infrastructure
docker-compose up             # Start Kafka, PostgreSQL, Conduktor, Schema Registry
docker-compose down           # Stop all services
```

## Ports

- Application: 8080
- Swagger UI: http://localhost:8080/swagger-ui/index.html
- Conduktor Console: 8090
- Schema Registry: 8081
- Kafka Broker: 9092
- PostgreSQL: 5433

## Architecture

### Request Flow
```
HTTP POST /echo → EchoController → EchoService → EchoProducer → Kafka topic-1
                                                                     ↓
PostgreSQL (kafka.messages) ← EchoService ← EchoConsumer ← topic-1
```

### Key Components

- **config/**: KafkaConfig (error handling with retry + DLT), KafkaTopicConfig (topic definitions), FlywayConfig
- **controller/**: Implements OpenAPI-generated `EchoApi` interface
- **producer/**: `EchoProducer` publishes to Kafka topics
- **consumer/**: `EchoConsumer` listens with `@KafkaListener`, logs headers
- **service/**: `EchoService` orchestrates publishing and processes consumed messages
- **kafka/**: Exception hierarchy - `RetryableProcessingException` (3 retries, 2s backoff) and `NonRetryableBusinessException` (sent to DLT)

### Code Generation

API interfaces and models are generated from `src/main/resources/openapi.yaml` during compilation. Generated code goes to `target/generated-sources/openapi/`.

### Database

- Flyway migrations in `src/main/resources/db/migration/`
- Schemas: `auth` (users), `kafka` (messages)
- Uses `application-local.yaml` profile for local PostgreSQL config

### Kafka Configuration

- Consumer group: "Group1"
- JSON serialization with trusted packages filter
- Topics defined in `KafkaTopicConfig`: topic-1, topic-2 (2 partitions each)
- KRaft mode (no Zookeeper)

## Development Notes

- Uses Lombok (`@Slf4j`, `@RequiredArgsConstructor`, `@Getter`, `@Setter`)
- Flyway set to clean database on validation failure in local profile - be aware when testing migrations
- Single consumer thread (concurrency=1)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slava-lu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/slava-lu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
