## mybatis-flex-gradle

> generateSchema: PUBLIC

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Gradle plugin for MyBatis Flex code generation. It wraps the MyBatis Flex codegen library to provide Gradle integration and includes Flyway database migration support.

Plugin ID: `io.github.fengshch.mybatis-flex-gradle-plugin`
Main class: `io.github.fengshch.mybatis.MyBatisFlexGradlePlugin`

## Build Commands

```bash
# Clean build
gradle clean

# Build the plugin
gradle :mybatis-flex-gradle-plugin:build

# Publish to local Maven repository
gradle :mybatis-flex-gradle-plugin:publishToMavenLocal

# Publish To Nexus Maven Repository
gradle :mybatis-flex-gradle-plugin:publishPluginMavenPublicationToNexus-releaseRepository    

# Run tests
gradle :mybatis-flex-gradle-plugin:test
```

## Project Structure

This is a composite build with the plugin in `mybatis-flex-gradle-plugin/` as an included build.

### Key Components

**Plugin Entry**: `MyBatisFlexGradlePlugin.java`
- Applies JavaBasePlugin
- Creates `mybatis` and `flyway` extensions
- **Loads configurations from mybatis-flex.yml** (if exists) before processing build.gradle configurations
- Registers generation and Flyway tasks
- Reads datasource config from YAML files (application.yml, mybatis-flex.yml)
- Supports Spring profiles: dev, test, default (via `-Pprofile=<name>` or PROFILE env var)

**Extension**: `MyBatisExtension.java`
- Provides `NamedDomainObjectContainer<GlobalConfigBuilder>` for multiple configurations
- Allows defining multiple generation profiles (e.g., "main", "secondary")

**Configuration Builders** (in `config/` package):
- `GlobalConfigBuilder` - main configuration container, holds all other builders
- `DataSourceConfigBuilder` - database connection settings
- `PackageConfigBuilder` - package structure and source directory
- `StrategyConfigBuilder` - table/column naming strategies
- `EntityConfigBuilder`, `MapperConfigBuilder`, `ServiceConfigBuilder`, `ServiceImplConfigBuilder`, `ControllerConfigBuilder`, `TableDefConfigBuilder`, `MapperXmlConfigBuilder` - code generation configs for each artifact type
- `FlywayConfigBuilder` - Flyway migration configuration
- `JavadocConfigBuilder` - javadoc generation settings
- `TemplateConfigBuilder` - custom template configuration

**Tasks**:
- `mybatisGenerate` (or `mybatis<Name>Generate` for named configs) - generates MyBatis Flex code
- Flyway tasks: `flywayClean`, `flywayBaseline`, `flywayMigrate`, `flywayUndo`, `flywayValidate`, `flywayInfo`, `flywayRepair` (prefixed with config name if not "main")

## Architecture

### Configuration Flow

1. Plugin creates `mybatis` extension with named configurations
2. **Loads configurations from `src/main/resources/mybatis-flex.yml`** if the file exists
3. In `afterEvaluate`, for each configuration:
   - Reads datasource config from YAML (supports spring.datasource structure)
   - Creates `mybatis<Name>Generate` task
   - Registers Flyway tasks
   - Adds custom source directories to Java source sets

### Code Generation Flow

1. `MyBatisGenerateTask` reads `GlobalConfigBuilder`
2. **Cleans the batis directory** (deletes the configured source directory to ensure fresh generation)
3. Creates HikariDataSource from `DataSourceConfigBuilder`
4. Builds `GlobalConfig` from all config builders
5. Detects database dialect from JDBC driver
6. Runs MyBatis Flex `Generator` to introspect schema and generate code
7. Generates: Entity, Mapper, Service, ServiceImpl, Controller, TableDef, MapperXml (based on enable flags)

### Datasource Configuration

The plugin reads datasource config from:
1. `src/main/resources/mybatis-flex.yml` (if exists)
2. `src/main/resources/application.yml` or profile-specific files (application-dev.yml, application-test.yml)

Expected YAML structure:
```yaml
spring:
  datasource:
    url: jdbc:...
    username: ...
    password: ...
    driverClassName: ... # or driver-class-name
```

For multiple configurations, use nested structure:
```yaml
spring:
  datasource:
    main:
      url: ...
    secondary:
      url: ...
```

### MyBatis Flex Configuration (mybatis-flex.yml)

You can define all MyBatis Flex configurations in `src/main/resources/mybatis-flex.yml` instead of in `build.gradle`. This provides a cleaner separation of configuration from build logic.

Example `mybatis-flex.yml`:
```yaml
spring:
  datasource:
    url: jdbc:h2:file:./data/h2db
    username: sa
    password: password
    driverClassName: org.h2.Driver

mybatis-flex:
  configurations:
    main:
      controllerGenerateEnable: true
      entityGenerateEnable: true
      mapperGenerateEnable: true
      serviceGenerateEnable: false
      serviceImplGenerateEnable: false
      tableDefGenerateEnable: true
      mapperXmlGenerateEnable: false

      packageConfig:
        sourceDir: src/batis/java
        basePackage: com.example.mybatis
        entityPackage: po
        mapperPackage: mapper
        servicePackage: service
        serviceImplPackage: service.impl
        controllerPackage: controller
        tableDefPackage: table
        mapperXmlPath: src/main/resources/mapper

      strategyConfig:
        tablePrefix: tb_
        logicDeleteColumn: deleted
        versionColumn: version
        generateForView: false
        generateSchema: PUBLIC
        unGenerateTables:
          - flyway_schema_history
          - CONSTANTS
          - ENUM_VALUES
          - INDEXES

      flyway:
        cleanDisabled: false
        locations:
          - classpath:db/migration
        baselineVersion: "1.0"
        baselineDescription: Initial baseline
        table: flyway_schema_history
        schemas:
          - public
```

**Note**: Configurations in `build.gradle` will override those in `mybatis-flex.yml` if both are present.

### Special Handling

- **H2 Database**: Relative paths (`jdbc:h2:file:./` or `jdbc:h2:file:../`) are converted to absolute paths based on project directory
- **Source Sets**: Custom source directories from `PackageConfigBuilder.sourceDir` are automatically added to the main source set
- **Directory Cleaning**: Before each code generation run, the batis directory (configured via `PackageConfigBuilder.sourceDir`, default: `src/batis/java`) is automatically deleted to ensure clean generation without leftover files from previous runs

## Supported Databases

- PostgreSQL (`org.postgresql.Driver`)
- MySQL (`com.mysql.cj.jdbc.Driver`)
- Oracle (`oracle.jdbc.driver.OracleDriver`)
- SQLite (`org.sqlite.JDBC`)
- H2 (for testing)

## Dependencies

Key dependencies (see `libs.versions.toml`):
- MyBatis Flex Codegen: 1.11.6
- Flyway: 12.0.2
- HikariCP: 7.0.2
- FreeMarker: 2.3.34 (for templates)
- JUnit Jupiter: 6.0.3

## Development Notes

- Java 25 source compatibility
- Uses Lombok for reducing boilerplate
- Test task configured with `failOnNoDiscoveredTests=false`
- Maven publishing to custom Nexus repository (requires NEXUS_USER and NEXUS_PASSWORD env vars)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fengshch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
