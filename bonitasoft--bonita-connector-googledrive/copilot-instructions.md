## bonita-connector-googledrive

> - **Name**: Bonita Google Drive Connector

# Bonita Google Drive Connector

## Project Overview

- **Name**: Bonita Google Drive Connector
- **Artifact**: `org.bonitasoft.connectors:bonita-connector-gdrive`
- **Version**: 1.0.0
- **Description**: Seven Bonita connectors for Google Drive operations: upload, download, create-folder, get, list, move, and delete.
- **License**: GPL-2.0
- **Tech stack**: Java 17, Maven, Bonita Engine 10.2.0, Google Drive API v3 (`google-api-services-drive`), Google Auth OAuth2 HTTP, Lombok, JUnit 5, Mockito, AssertJ, jqwik (property-based testing), WireMock, Testcontainers

## Build Commands

```bash
# Full build including unit + integration tests (default goal)
./mvnw clean verify

# Skip tests
./mvnw clean verify -DskipTests

# Unit tests only (Surefire)
./mvnw test

# Integration tests only (Failsafe)
./mvnw verify -DskipTests=false

# Package connector ZIPs
./mvnw clean package

# Deploy to Maven Central (requires GPG key)
./mvnw clean deploy -Pdeploy
```

The build produces one ZIP assembly per connector operation under `target/`, each deployable into Bonita Studio.

## Architecture

### Class hierarchy

```
AbstractConnector (bonita-common)
  └── AbstractGDriveConnector              # Shared lifecycle: validate → connect → execute → disconnect
        ├── UploadFileConnector
        ├── DownloadFileConnector
        ├── CreateFolderConnector
        ├── GetFileConnector
        ├── ListFilesConnector
        ├── MoveFileConnector
        └── DeleteFileConnector

GDriveConfiguration   # Holds serviceAccountKeyJson and operation-specific params (built per connector)
GDriveClient          # Wraps Google Drive API; created in connect(), torn down in disconnect()
GDriveException       # Domain exception for Google Drive errors
RetryPolicy           # Retry logic for transient API failures
```

### Key patterns

- **Template method**: `AbstractGDriveConnector.executeBusinessLogic()` handles success/error output; subclasses implement `doExecute()`.
- **Configuration builder**: Each subclass implements `buildConfiguration()` to read its own input parameters into a `GDriveConfiguration`.
- **Lifecycle hooks**: `connect()` instantiates `GDriveClient`; `disconnect()` nulls it out.
- **Non-throwing execution**: Exceptions from `doExecute()` are caught and written to `success`/`errorMessage` output parameters rather than re-thrown, allowing the process to continue.
- **Lombok**: `@Slf4j` and `@Data`/`@Builder` are used to reduce boilerplate.
- **Assembly**: `src/assembly/` contains descriptors that generate one ZIP per connector.

## Testing Strategy

- **Unit tests** (`*Test.java`): JUnit 5 + Mockito; mock `GDriveClient` and Google API objects.
- **Property-based tests** (`*PropertyTest.java`): jqwik; verify connector behaviour over generated inputs.
- **Integration tests** (`*IntegrationTest.java`): WireMock stubs for Google Drive API HTTP responses; no real Google credentials needed.
- **End-to-end IT** (`GDriveConnectorIT.java`): Testcontainers + `bonita-java-client`; requires a running Bonita instance (skipped by default in CI unless configured).
- Coverage via JaCoCo, report at `target/site/jacoco/`.

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>

[optional body]

[optional footer(s)]
```

Common types: `feat`, `fix`, `chore`, `refactor`, `test`, `docs`, `ci`.

Examples:
```
feat: add move-file connector
fix: handle 404 response from Drive API gracefully
chore: bump google-api-services-drive to v3-rev20260220-2.0.0
```

## Release Process

1. Remove `-SNAPSHOT` from `version` in `pom.xml` (root).
2. Update connector definition versions (e.g. `get-file.def.version`) if connector definitions changed.
3. Commit: `chore: release X.Y.Z`.
4. Tag: `git tag X.Y.Z`.
5. Deploy to Maven Central: `./mvnw clean deploy -Pdeploy` (requires GPG key and Central credentials in `~/.m2/settings.xml`; JFrog Artifactory repo configured as `bonitasoft-releases`).
6. Push tag: `git push origin X.Y.Z`.
7. Bump to next `-SNAPSHOT` and commit: `chore: prepare next development iteration`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bonitasoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
