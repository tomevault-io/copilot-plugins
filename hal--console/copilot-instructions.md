## console

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HAL is the WildFly / JBoss EAP management console. It is a client-side GWT application written entirely in Java 11 that gets transpiled to JavaScript. It communicates with WildFly management endpoints using the DMR (Dynamic Model Representation) protocol over HTTP.

## Build Commands

```bash
# Full build (includes GWT compilation - slow, ~5-10 min)
./mvnw verify

# Quick build skipping GWT compilation (use for Java-only changes)
./mvnw verify -P skip-gwt

# Even faster: skip GWT + skip tests
./mvnw install -Dquickly

# Run a single test
./mvnw test -pl <module> -Dtest=<TestClass>
# Example: ./mvnw test -pl dmr -Dtest=ResourceAddressTest

# Format source code (license headers, code formatting, import sorting)
./format.sh

# Validate source code (enforcer, checkstyle, license, formatting, imports)
./validate.sh

# GWT dev mode (Super Dev Mode) for live reloading during UI development
# Requires a running WildFly instance
cd app && mvn gwt:devmode
```

### Maven Profiles

| Profile | Activation | Purpose |
|---------|-----------|---------|
| `skip-gwt` | `-P skip-gwt` | Skip GWT compilation (CI uses this) |
| `quick-build` | `-Dquickly` | Skip GWT + skip integration tests |
| `prod` | `-P prod` | Production GWT compilation (no draft compile) |
| `native` | `-P native` | Build native binary via Quarkus |
| `theme-hal` | Active by default | HAL community theme |
| `theme-wildfly` | `-P theme-wildfly` | WildFly theme |
| `theme-eap` | `-P theme-eap` | EAP theme |

## Architecture

### Module Dependency Graph (top-down)

```
app (hal-console) ─── GWT entry point, all Presenters and Views
 ├── core ─── Business logic, CRUD operations, Finder, MVP framework
 │    ├── ballroom ─── UI components (forms, tables, dialogs, PatternFly widgets)
 │    │    └── dmr ─── DMR model (ModelNode, Operation, Composite, Dispatcher)
 │    │         ├── config ─── Configuration and environment settings
 │    │         │    └── js ─── JavaScript interop (clipboard, JSON, etc.)
 │    │         │         └── resources ─── I18n constants, CSS, IDs
 │    │         │              └── spi ─── Annotations (@Column, @MbuiView, @Requires, etc.)
 │    │         └── flow ─── Async task sequencing (promises, sequential/parallel tasks)
 │    ├── meta ─── Metadata, AddressTemplate, security, resource descriptions
 │    └── db ─── PouchDB wrapper for client-side storage
 ├── processors ─── Annotation processors (compile-time code generation)
 └── standalone ─── Quarkus server for standalone mode (no WildFly needed)
```

### Key Architectural Patterns

**MVP (Model-View-Presenter)**: Uses GWTP framework. Each screen has a `*Presenter` (logic, DMR calls) and a `*View` (DOM rendering). Presenters are bound via GIN dependency injection in `ConsoleModule`.

**MBUI (Model-Browser UI)**: Views annotated with `@MbuiView` have their UI generated from companion XML files (`*.mbui.xml`) by annotation processors at compile time. 36 views use this pattern. The processor generates `Mbui_*View` classes.

**Finder Pattern**: Column-based navigation (like macOS Finder). Columns are registered via `@Column` annotations and managed by `FinderColumn`/`Finder` in `core/finder/`.

**DMR Dispatch**: All WildFly communication goes through `Dispatcher` using `Operation` and `Composite` objects. `ModelNode` is the universal data type (mirrors WildFly's management model).

**AddressTemplate**: Parameterized resource addresses (e.g., `/{selected.host}/subsystem=datasources/data-source=*`) resolved at runtime via `StatementContext`.

### Package Organization (in `app/src/main/java/org/jboss/hal/client/`)

- `configuration/` - Configuration subsystem screens (datasources, messaging, EE, etc.)
- `runtime/` - Runtime views (server status, deployments, logging)
- `accesscontrol/` - RBAC management
- `deployment/` - Deployment management
- `bootstrap/` - Application startup (HalPreBootstrapper, HalBootstrapper)
- `homepage/` - Landing page
- `tools/` - Macro recording, management model browser

## Code Style

- Code formatting and import ordering enforced by Maven plugins; run `./format.sh` before committing
- License headers on all files are enforced by `license-maven-plugin`
- Checkstyle rules in `build-config/src/main/resources/etc/`

## Branching

- `main` - Active development
- `develop` - PR target branch
- `3.6.x`, `3.3.x`, etc. - Maintenance branches for older WildFly/EAP versions

## Issue Tracking

Bugs and features tracked at https://issues.jboss.org/browse/HAL (JIRA, project key: HAL)

---
> Source: [hal/console](https://github.com/hal/console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
