## sirk-taxonomy-commons

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a common base library for Sirktek's RDF-S taxonomy projects. It provides shared model classes, an abstract RDF-S loader, and a service layer that is extended by domain-specific taxonomy libraries (furniture-taxonomy, logistics-taxonomy).

## Common Commands

### Build and Test
```bash
# Using system Maven
mvn compile          # Compile source code
mvn test             # Run unit tests
mvn package          # Build JAR artifact
mvn install          # Install to local Maven repository
mvn clean            # Clean build artifacts

# Using Maven wrapper (recommended)
./mvnw compile       # Compile source code
./mvnw test          # Run unit tests
./mvnw package       # Build JAR artifact
./mvnw install       # Install to local Maven repository
./mvnw clean         # Clean build artifacts
```

### Running Single Tests
```bash
# Run a specific test class
mvn test -Dtest=TaxonomyServiceTest

# Run a specific test method
mvn test -Dtest=TaxonomyServiceTest#testLoadBaseTaxonomy
```

## Architecture

### Layered Structure
- **Model Layer** (`model/`): Core data structures (`CategoryInfo`, `TaxonomyTree`, `PropertyDefinition`)
- **Loader Layer** (`loader/`): Abstract `RdfsTaxonomyLoader` using Apache Jena
- **Service Layer** (`TaxonomyService`): Main public API with thread-safe caching

### Extension Pattern
Domain-specific taxonomy libraries extend the abstract `RdfsTaxonomyLoader`:

1. Extend `RdfsTaxonomyLoader` and override:
   - `getNamespace()`: Return taxonomy namespace URI
   - `getResourcePath()`: Return path to RDF-S Turtle file
2. Create domain service extending `TaxonomyService`
3. Create domain-specific property type detection if needed

### Key Files
- `model/CategoryInfo.java` - Taxonomy category representation
- `model/TaxonomyTree.java` - Complete taxonomy tree
- `model/PropertyDefinition.java` - RDF property definition
- `loader/RdfsTaxonomyLoader.java` - Abstract RDF-S loader
- `TaxonomyService.java` - Service with caching

## Development Notes

### Technology Stack
- Java 17
- Apache Maven 3.9+
- Apache Jena 5.5.0 for RDF processing
- Lombok 1.18.36 for code generation
- JUnit Jupiter 5.11.3 for testing

### Threading and Caching
- Uses thread-safe singleton pattern for taxonomy loading
- Double-checked locking for cache initialization
- Cache invalidation via `reloadBaseTaxonomy()`

### Language Support
- English URIs and class names for API compatibility
- Bilingual labels (English/Norwegian) stored in RDF-S for frontend consumption
- Frontends handle translation - library provides English class names

### Dependent Projects
This commons library is used by:
- **sirk-furniture-taxonomy**: Furniture classification taxonomy
- **sirk-logistics-taxonomy**: Logistics and location taxonomy

When making changes, ensure backward compatibility and test with all dependent projects.

### CI/CD Pipeline
- **Pull Requests**: PMD analysis + full test suite
- **Main Branch**: Auto-build, version tagging (1.{build_number}), deploy to Maven Central
- **Maven Wrapper**: Available for development (./mvnw, ./mvnw.cmd)
- **Versioning**: Uses `${revision}` property for dynamic versioning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sirktek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
