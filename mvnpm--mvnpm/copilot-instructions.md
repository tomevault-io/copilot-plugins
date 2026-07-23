## mvnpm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mvnpm** (Maven NPM) is a Quarkus-based web service that acts as a Maven repository facade on top of the NPM Registry. It converts NPM packages into Maven-compatible artifacts (JARs, POMs, etc.) so Java/Maven/Gradle projects can consume NPM packages as standard dependencies. It also handles syncing packages to Maven Central.

## Build and Development Commands

```bash
# Full build
./mvnw install

# Dev mode (requires PostgreSQL via Quarkus Dev Services)
./mvnw quarkus:dev

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=VersionConverterTest

# Run a single test method
./mvnw test -Dtest=VersionConverterTest#testCaretVersion

# Format code (runs automatically during build)
./mvnw formatter:format impsort:sort
```

## Code Formatting

The project uses Quarkus-style formatting enforced by two Maven plugins that run during build:
- **formatter-maven-plugin**: Eclipse-based Java formatting (`eclipse-format.xml` config from `quarkus-ide-config`)
- **impsort-maven-plugin**: Import sorting with groups `java., javax., jakarta., org., com.` and unused import removal

## Architecture

### Core Flow: NPM → Maven

1. **`MavenRepositoryApi`** (`/maven2/org/mvnpm/...`) — REST endpoint that mimics a Maven repository. Handles requests for POMs, JARs, sources, javadoc, tgz, metadata, and their hashes/signatures.

2. **`NpmRegistryFacade`** / **`NpmRegistryClient`** — REST client to the NPM Registry (`registry.npmjs.org`). Fetches project metadata and package info with caching and fault tolerance.

3. **`PackageCreator`** — Orchestrates creating Maven artifacts from NPM packages. Uses a local file cache; creates on-demand when not cached.

4. **Creator services** (`io.mvnpm.creator.type.*`):
   - `TgzService` — Downloads the NPM tarball
   - `JarService` — Repackages the tgz contents into a JAR
   - `PomService` — Generates a Maven POM from package.json
   - `MetadataService` — Generates maven-metadata.xml
   - `HashService` / `AscService` — Creates SHA1, MD5, and PGP signatures
   - `SourceService` / `JavaDocService` — Creates source and javadoc JARs

5. **Maven Central sync** (`io.mvnpm.mavencentral.sync.*`):
   - `CentralSyncService` — Manages sync lifecycle and status tracking
   - `BundleCreator` — Creates the upload bundle for Maven Central
   - `CentralSyncItem` — JPA entity tracking sync state per artifact version (stages: NONE → PACKAGING → UPLOADING → RELEASED)
   - `ContinuousSyncService` — Scheduled monitoring for new NPM versions

### NPM ↔ Maven Name Mapping

`NameParser` and `Name` handle the bidirectional mapping:
- Non-scoped: `lit` → `org.mvnpm:lit`
- Scoped: `@hotwired/stimulus` → `org.mvnpm.at.hotwired:stimulus`

### Version Conversion

`VersionConverter` translates NPM semver ranges (tilde, caret, hyphen, X-ranges, operators) to Maven version range syntax.

### Composites

`io.mvnpm.creator.composite.*` — Supports creating composite packages that aggregate multiple NPM packages into one Maven artifact, configured via GitHub.

### Frontend

The UI is a Lit-based SPA using Vaadin web components, bundled via `quarkus-web-bundler`. Source is in `src/main/resources/web/app/` (TypeScript). The backend serves SPA routes via `SPARouting`.

### Database

PostgreSQL with Hibernate ORM Panache. Dev/test mode uses Quarkus Dev Services (auto-provisioned DB) with `drop-and-create`. Production uses `update` strategy.

### Key Configuration

- `mvnpm.local-user-directory` — Where artifacts are cached on disk (`target` in dev, `/opt/mvnpm` in prod)
- `mvnpm.local-m2-directory` — Local Maven cache directory name
- `mvnpm.mavencentral.autorelease` — Auto-release to Central (disabled in dev/test)
- Cron schedules for periodic version checks and error retries

---
> Source: [mvnpm/mvnpm](https://github.com/mvnpm/mvnpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
