## myhab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

myHAB (My Home Automation Backend) — a full-stack home automation platform. Grails 6.1/Groovy backend + Vue 3/Quasar PWA frontend. Manages IoT devices (ESP32, MEGAD), solar inverters (Huawei), heat pumps (NIBE), weather data, and automation scenarios via MQTT, GraphQL, and WebSocket.

## Build & Run Commands

### Backend (Grails/Gradle)
```bash
./gradlew bootRun                        # Run Grails backend (port 8181)
./gradlew test                           # Run all server tests (Spock)
./gradlew server-core:test               # Run server-core tests only
./gradlew server-core:test --tests "org.myhab.services.StatisticsServiceSpec"  # Single test class
./gradlew assembleServerAndClient        # Build combined JAR (server + client)
./gradlew buildImage                     # Build Docker image
./gradlew clean copyClientResources assembleServerAndClient createDockerfile  # Full CI build
```

### Frontend (Vue 3 + Quasar)
```bash
cd client/web-vue3
yarn install                             # Install dependencies
yarn serve                               # Dev server on port 10002
yarn pwa:dev                             # PWA dev mode on port 10002
yarn build                               # Production SPA build
yarn pwa:build                           # Production PWA build
yarn lint                                # ESLint
yarn format                              # Prettier
```

### Gradle shortcuts (from root)
```bash
./gradlew serve                          # Run SPA dev server
./gradlew servePWA                       # Run PWA dev server
```

## Architecture

### Multi-module Gradle project
- `:server-core` — Main Grails 6.1 application (controllers, domain, services, jobs)
- `:server-config` — Configuration key constants and provider interface
- `:server-rules` — Business rules engine (heating, lighting facts)
- `:web-vue3` — Vue 3 + Quasar frontend

### Backend structure (`server/server-core/grails-app/`)
- **controllers/** — REST endpoints + `UrlMappings.groovy` for routing. GraphQL is the primary API.
- **domain/** — GORM domain classes (~80). Key hierarchies: `device/` (Device, DevicePort, DevicePeripheral, Cable), `infra/` (Zone, Layer), `job/` (Job, Scenario, EventDefinition, CronTrigger), `auth/` (User, Role, AccessToken), `events/` (MQTTTopic)
- **services/** — Business logic. Key subsystems: `graphql/` (schema factory, fetchers for Query/Mutation), `async/mqtt/` (MQTT integration via Spring Integration), `async/socket/` (WebSocket), `dsl/` (automation DSL), `telegram/` (bot handler)
- **jobs/** — Quartz scheduled jobs (13 jobs). Intervals configured in `application.yml` under `myhab.job.*`
- **conf/application.yml** — Server port (8181), Quartz job intervals, CORS origins, database config

### Frontend structure (`client/web-vue3/src/`)
- **boot/** — App initialization (axios, Apollo GraphQL client, i18n, PWA, error handler)
- **pages/** — Route-level components. `public/` contains unauthenticated pages
- **components/** — Reusable Vue components
- **graphql/** — GraphQL query/mutation definitions
- **composables/** — Vue 3 composables
- **_services/** and **_helpers/** — Service layer and utilities

### Key communication patterns
1. **Device ↔ Server**: MQTT via Spring Integration (`MqttTopicService` routes messages, `MqttPublishGateway` publishes)
2. **Client ↔ Server**: GraphQL (Apollo Client) for queries/mutations; WebSocket (STOMP) for real-time updates
3. **Scheduled sync**: Quartz jobs poll external APIs (Huawei solar, NIBE heat pump, Open-Meteo weather) and publish state via MQTT

### GraphQL schema
Defined in `server/server-core/src/main/resources/schema.graphqls`. Custom fetchers live in `services/graphql/fetchers/` (Query.groovy, Mutation.groovy). GORM domain classes are auto-registered via `GORMSchemaRegistry`.

### Event system
Events flow through named topics (e.g., `EVT_LIGHT`, `EVT_GATE`) with parameters p0–p6. `EventDefinition` and `EventSubscription` domain classes define event routing. Scenarios can trigger on event patterns.

## Tech Stack & Requirements

- **Java 17** (required — Temurin recommended)
- **Node.js 18+** with Yarn (frontend)
- **Grails 6.1.0** / Groovy 3.0.25 / GORM 7.3.4
- **PostgreSQL** database
- **Vue 3** / Quasar 2.18 / Apollo Client 3.14
- **Spring Security Core 6.0** with OAuth 2.0 token auth
- **Hazelcast 5.3** for caching

## Testing

Backend tests use **Spock** framework:
- Unit tests: `server/server-core/src/test/groovy/`
- Integration tests: `server/server-core/src/integration-test/groovy/`
- Code style: CodeNarc rules in `server/server-core/config/codenarc/rules.groovy`

Frontend linting: ESLint (`.eslintrc.js`) + Prettier (`.prettierrc.js`)

## CI/CD

GitHub Actions (`.github/workflows/gradle.yml`): triggers on push/PR to master or beta. Builds combined JAR, creates Docker image (`eclipse-temurin:17-jre` base), pushes to DockerHub as `kirpi4ik/myhab:{version}-{sha}`.

## Configuration

- Backend config: `server/server-core/grails-app/conf/application.yml` (job intervals, CORS, DB)
- Runtime config: `server/server-core/grails-app/conf/runtime.groovy`
- Spring beans: `server/server-core/grails-app/conf/spring/resources.groovy`
- Frontend config: `client/web-vue3/quasar.config.js` (build modes, plugins, dev proxy)
- Frontend env: `client/web-vue3/src/.env`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kirpi4ik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kirpi4ik)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
