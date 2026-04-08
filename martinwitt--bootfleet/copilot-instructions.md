## bootfleet

> NEVER CREATE A SUMMARY FILE AFTER UR WORK IS DONE.

# BootFleet AI Agent Guide

NEVER CREATE A SUMMARY FILE AFTER UR WORK IS DONE.
NEVER RUN MAVEN CLEAN

## Architecture Overview
BootFleet is a Maven multi-module monorepo containing independent Spring Boot microservices:
- **config-reloader**: Kubernetes config watcher with web UI for ConfigMap/Secret stats (hexagonal architecture: domain/model/port/service, infrastructure)
- **maven-version-mcp**: MCP server for Maven artifact version queries using direct Maven Central API calls
- **url-cleaner**: Web app to remove tracking parameters (utm_*, fbclid, etc.) from URLs
- **mail-summary**: AI-powered email summarization service (Spring AI integration)

Modules use shared parent POM with Java 25, Spring Boot 4.x, and Spotless formatting.

## Key Workflows
- **Build all modules**: `mvn clean install` from root (runs spotless check in verify phase)
- **Format code**: `mvn spotless:apply` (Google Java Format AOSP style, removes unused imports, trims whitespace)
- **Run service**: `mvn spring-boot:run` in module directory (e.g., config-reloader uses Fabric8 Kubernetes client)
- **Native build**: `mvn -Pnative native:compile` (url-cleaner has GraalVM config)
- **Test**: `mvn test` (standard JUnit, some AOT metadata generation)

## Project Conventions
- **Package structure**: `io.github.martinwitt.{module}` (e.g., `io.github.martinwitt.configreloader`)
- **Controllers**: Thymeleaf templates in `src/main/resources/templates/` (e.g., config-reloader's `stats.html` fragment)
- **Configuration**: `application.yaml` in `src/main/resources/` (e.g., module-specific settings)
- **Dependencies**: Managed in parent POM, use BOMs for Spring Boot/Spring Cloud
- **Crypto/Security**: BouncyCastle in config-reloader for certificate handling
- **Kubernetes integration**: Fabric8 client with Spring Cloud Kubernetes (config-reloader)
- **AI integration**: Spring AI snapshots for mail-summary

## Integration Points
- **Kubernetes API**: config-reloader watches ConfigMaps/Secrets and dependent Deployments
- **Maven Central**: maven-version-mcp queries artifact versions via HTTP
- **Web UIs**: All web modules serve HTMX-style fragments (e.g., url-cleaner's form POST redirects with flash attributes)

## Examples
- Add new tracking param to remove: Edit `TRACKING_PARAMS` set in `UrlController.java`
- Query Maven versions: Use MCP tools like `maven-get-latest-version` with groupId/artifactId
- Watch K8s resource: Implement `ConfigResourceRepository` (in-memory default, extend for persistence)</content>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MartinWitt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MartinWitt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
