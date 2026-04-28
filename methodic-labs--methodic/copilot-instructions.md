## methodic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Methodic is a **Gradle multi-project superproject** that aggregates related subprojects via **git submodules**. The subprojects are: `rhizome`, `rhizome-client`, `chronicle-api`, `chronicle-server`, and `chronicle-web` (plus `chronicle` mobile, currently disabled).

## Build Commands

```bash
# Build everything
./gradlew build

# Build a specific subproject
./gradlew :rhizome:build
./gradlew :chronicle-server:build

# Run tests for a specific subproject
./gradlew :rhizome:test
./gradlew :chronicle-api:test

# Run a single test class
./gradlew :chronicle-server:test --tests "com.openlattice.chronicle.SomeTest"

# Clean build
./gradlew clean build

# Publish libraries (chronicle-api, rhizome, rhizome-client)
./gradlew :chronicle-api:publish :rhizome:publish :rhizome-client:publish
```

CI skips chronicle-server SpotBugs and tests: `gradle build -x :chronicle-server:spotbugsMain -x :chronicle-server:spotbugsTest -x :chronicle-server:test`

### chronicle-web (Frontend)

```bash
cd chronicle-web

# Install dependencies
npm install

# Dev server
npm run app

# Production build
npm run build:prod

# Lint (CSS + JS)
npm run lint

# Tests
npm test
npm run test:watch
```

## Architecture

### Subproject Dependency Graph

```
chronicle-server  →  chronicle-api  →  rhizome-client
                  →  rhizome        →  rhizome-client

chronicle-web (frontend)  →  chronicle-server (API backend)
```

- **rhizome** / **rhizome-client**: Foundational framework. Provides embedded Jetty server bootstrap (`RhizomeApplicationServer`, `JettyContainerPod`), Hazelcast integration, HikariCP connection pooling, Auth0 security, and Spring MVC wiring. rhizome-client is the lightweight API/client counterpart.
- **chronicle-api**: API definitions, models, and Retrofit client interfaces. Targets **Java 17** (Android compatibility). All other subprojects target **Java 21**.
- **chronicle-server**: The main application server. Entry point: `com.openlattice.chronicle.ChronicleServer`. Contains controllers, services, JDBI-based data access, Hazelcast mapstores, and WebSocket support.
- **chronicle-web**: React frontend SPA. Uses Redux + Redux-Saga for state management, styled-components for styling, Flow for type checking, and Webpack 5 for bundling. ESLint follows airbnb config with 2-space indentation and 120-char max line length. Tests use Jest + Enzyme.

### Development Mode

`gradle.properties` sets `developmentMode=true`, which causes the shared Gradle scripts to substitute published Maven artifacts (`com.getmethodic` group) with local project dependencies. This allows cross-subproject refactoring without publishing.

### Shared Gradle Configuration

All subproject `build.gradle` files (Groovy DSL) apply `gradles/methodic.gradle`, which provides:
- Centralized dependency version catalog (all library versions defined in one place)
- Dependency substitution logic for development mode
- JaCoCo coverage, Checkstyle, and SpotBugs configuration
- Publishing logic is in `gradles/publish.gradle` (snapshots from `develop`, releases from `main`/`release/*`; versioning via `git describe --tags`)

### Languages

Java and Kotlin coexist in every subproject. Newer code is predominantly Kotlin. Kotlin compiler flags include `-Xjvm-default=all`.

### Key Technology Stack

- **Web**: Spring MVC (not Boot) with embedded Jetty 12 (ee8)
- **Database**: PostgreSQL via JDBI3 + HikariCP; Redshift and Snowflake connectors in chronicle-server
- **Distributed cache**: Hazelcast 5.5 with custom `SelfRegisteringStreamSerializer` pattern
- **Serialization**: Jackson 2.19+ with extensive modules (Guava, Kotlin, Smile binary, JodaTime, JSR310)
- **Auth**: Auth0 + Spring Security
- **HTTP clients**: Retrofit2 3.0
- **AWS**: SDK v2 (S3, EC2, SNS) in rhizome/chronicle-server; SDK v1 in rhizome-client
- **Logging**: SLF4J 2.0 → Log4j2

### Spring Wiring Pattern

Spring configuration uses a "Pod" pattern — `@Configuration` classes named `*Pod` (e.g., `ChronicleServerServletsPod`, `JettyContainerPod`) that define beans and are composed together. There is no Spring Boot auto-configuration.

### Testing

- **JUnit 4** with Mockito 1.x
- Integration tests spin up a full `BaseRhizomeServer` (embedded Jetty + Spring context) and interact via Retrofit-backed clients
- Tests require extensive `--add-opens`/`--add-exports` JVM flags (configured in Gradle)

## Submodule Workflows

```bash
# Clone with submodules
git clone ssh://git@github.com/methodic-labs/methodic.git --recurse-submodules

# Check status across all submodules
git submodule foreach 'git status'

# Update all submodules to a branch
git submodule foreach "git checkout develop"
git submodule foreach "git pull || :"
```

## Package Namespaces

- `com.geekbeast.*` — rhizome and rhizome-client
- `com.openlattice.*` — chronicle-api and chronicle-server

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/methodic-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
