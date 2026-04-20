## sheets

> PostgreSQL Usage Rules (Spring Web & Coroutines) - Specific Directives


PostgreSQL Usage Rules (Spring Web & Coroutines) - Specific Directives
Purpose
This document defines highly specific, mandatory rules for PostgreSQL interaction in Spring Web/Kotlin Coroutines services. Directives cover precise database access, robust migration, and consistent build configurations for strict cascade injection.

Database Interaction Principles
For all PostgreSQL interactions, these principles MUST be rigorously adhered to:
USE ALWAYS: id("org.flywaydb.flyway") version "9.16.1"

Jooq for Database API Calls (Synchronous & Coroutine Integration):

Mandatory Jooq Usage: All database queries (SELECT), DML (INSERT, UPDATE, DELETE), and dynamic DDL MUST use Jooq exclusively for consistent, type-safe data access.

Type-Safe Code Generation: Jooq's code generation MUST produce type-safe Kotlin classes (records, tables, POJOs, DAOs) as the primary interface, eliminating runtime SQL errors.

Synchronous Jooq Library: The synchronous (blocking) Jooq API MUST be used.

Coroutine Integration: All synchronous Jooq calls MUST be explicitly wrapped within withContext(Dispatchers.IO) to offload blocking I/O from the main event loop. This is critical for responsiveness in reactive (Spring WebFlux) and coroutine-based applications; failure leads to thread starvation.

Example:

suspend fun findUserById(userId: UUID): UserRecord? = withContext(Dispatchers.IO) {
    dslContext.selectFrom(USERS)
              .where(USERS.ID.eq(userId))
              .fetchOneInto(UserRecord::class.java)
}

Error Handling: Within withContext(Dispatchers.IO) blocks, try-catch MUST manage database-specific errors (SQLException, DataAccessException), mapping them to domain-specific exceptions.

DSLContext Management: DSLContext MUST be properly injected and managed (e.g., via Spring DI). Ensure DSLContext is configured with correct ConnectionProvider and ExecuteListenerProvider.

Transaction Management: Transactions MUST be managed programmatically via Jooq's API (e.g., dslContext.transaction { ... }) or declaratively with Spring's @Transactional on suspending functions. Ensure compatibility with coroutines and Dispatchers.IO.

Raw SQL Prohibition: Direct raw SQL execution is STRICTLY PROHIBITED, including dynamic string concatenation. Exceptions ONLY for highly complex, performance-critical queries Jooq cannot express, and ONLY with explicit architectural approval and documentation outlining rationale and SQL injection mitigation.

Flyway for Database Migrations (Versioned & Idempotent):

Exclusive Migration Tool: Database schema changes (DDL) and initial/corrective data migrations (DML) MUST be managed exclusively using Flyway. Manual schema changes on deployed environments are STRICTLY PROHIBITED.

Versioned SQL Scripts: All schema/data changes MUST be defined as versioned SQL scripts. Naming convention MUST strictly follow V<VERSION>__<DESCRIPTION>.sql.

Migration Script Content: Each script SHOULD focus on a single logical change. Scripts MUST be idempotent where possible. DDL changes MUST be backward-compatible for zero-downtime deployments; avoid dropping columns/tables if previous app version relies on them. Avoid DDL statements inside transactions unless explicitly supported by PostgreSQL and Flyway.

Migration Directory: All migration scripts MUST reside in src/main/resources/db/migration (or configured flyway.locations).

Baseline and Repair: Flyway's baseline MUST be used ONLY for initializing Flyway on existing non-Flyway managed databases. repair MUST be used with extreme caution and strict supervision for checksum mismatches.

Review Process: All new/modified Flyway scripts MUST undergo peer review before merging to main or deployment branches. Review MUST include verification of SQL correctness, idempotency, and backward compatibility.

Build Configuration (build.gradle)
The build.gradle file MUST be configured with these specific setups:

Flyway Plugin Integration:

The org.flywaydb.flyway plugin MUST be applied at the top.

Mandatory Configuration: Flyway parameters MUST be explicitly defined in build.gradle or via application.properties/application.yml: flyway.url, flyway.user, flyway.password (from env vars), flyway.schemas (e.g., ['public']), flyway.locations (MUST point to classpath:db/migration). flyway.cleanDisabled = true SHOULD be set in production profiles.

Tasks: The flywayMigrate task SHOULD be explicitly run during deployment pipelines before application starts.

Jooq Generator Setup:

The nu.studer.jooq plugin MUST be applied.

Generator Configuration: The jooq block MUST generate Kotlin classes: jooq.version (MUST match Jooq library version), jooq.edition (MUST be JooqEdition.OSS), generationTool.jdbc (MUST specify correct driver, url, user, password matching target DB, distinct from runtime for security). generator.name (MUST be org.jooq.codegen.KotlinGenerator). database.inputSchema (MUST specify schema (e.g., 'public')). generate (MUST include pojos = true, daos = true, relations = true, records = true, routines = true, kotlinSetterGetters = true). target.packageName (MUST follow project conventions). target.directory (MUST be build/generated-src/jooq/main). Custom Type Mappings: If custom PostgreSQL types (enums, JSONB) are used, custom Jooq converters/bindings SHOULD be configured.

Automated Generation: The Jooq generator MUST run automatically as part of the build task (e.g., depending on compileKotlin).

Clean Task: cleanJooq SHOULD be used to remove generated Jooq classes before a clean build.

Testing Guidelines
No Test Cases for Database Layer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rajat9garg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
