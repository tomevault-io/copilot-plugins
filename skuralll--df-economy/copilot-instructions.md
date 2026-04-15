## df-economy

> Economic system for Minecraft Dragonfly server

# DF Economy Project

Economic system for Minecraft Dragonfly server

## About Dragonfly
- Asynchronous Minecraft: Bedrock Edition server written in Go
- Extensibility-focused design, used as a library
- Struct-based command system with reflection-based argument parsing
- Client-side command integration (auto-completion, validation)

## Command System Basics
- Implement `cmd.Runnable` interface
- Struct fields become command parameters
- Use `cmd.SubCommand` to define subcommands
- Use `cmd.Optional[T]` for optional arguments
- Commands registered via `cmd.Register()` with `cmd.New()`
- Error handling through `cmd.Output` and direct player messaging

## Project Structure
```
df-economy/
├── dragonfly/commands/     # Command implementations
│   ├── base.go            # BaseCommand with shared utilities
│   ├── register.go        # Command registration
│   ├── economy.go         # Main help command
│   ├── balance.go         # Balance display (supports optional target)
│   ├── set.go             # Balance setting (configurable command)
│   ├── top.go             # Rankings with pagination
│   └── pay.go             # Money transfer between players
├── economy/               # Domain and business logic
│   ├── domain.go          # EconomyEntry domain model
│   ├── config/            # Configuration management
│   │   └── config.go      # Economy configuration struct with multi-RDBMS and command control
│   └── service/           # Business logic layer
│       ├── service.go     # EconomyService with validation
│       └── errors.go      # Service-specific error types
├── internal/db/           # Database abstraction
│   ├── db.go              # DB interface
│   ├── factory.go         # Database factory for multi-RDBMS support
│   ├── gorm.go            # GORM implementation with multi-RDBMS support
│   ├── model.go           # Account database model
│   └── errors.go          # Database-specific error types
├── cmd/demo/              # Demo server implementation
│   └── main.go            # Server setup with economy integration
└── config.toml            # Dragonfly server configuration
```

## Architecture Design
- **Domain Layer**: Clean `EconomyEntry` struct representing core business objects
- **Service Layer**: `EconomyService` handles business logic, validation, and configuration
- **Database Layer**: Interface-based design with GORM ORM implementation supporting multiple RDBMSs
- **Command Layer**: Dragonfly commands with async execution
- **Error Handling**: Custom error types with user-friendly messages, layered by responsibility
- **Configuration**: Configurable database type, DSN, default balance, and command availability settings
- **Factory Pattern**: Database factory for multi-RDBMS support (SQLite, MySQL, PostgreSQL)
- **Async Processing**: Non-blocking command execution with timeout handling

## Command Implementation Patterns
- **BaseCommand**: Shared utilities for player validation, async execution, UUID resolution
- **Async Execution**: All database operations run asynchronously with 5-second timeout
- **Error Messaging**: Consistent error handling with colored chat messages
- **Immediate Feedback**: Commands provide instant feedback before async processing

## Database Operations
- **Multi-RDBMS Support**: SQLite, MySQL, and PostgreSQL support via GORM ORM
- **Factory Pattern**: Database factory (`NewDB`) creates appropriate dialector based on configuration
- **Account Model**: Database schema with UUID, Name, Balance fields using GORM tags
- **Balance Management**: Get, Set, Transfer operations with atomic transactions
- **User Registration**: Automatic user registration on first join with configurable default balance
- **Leaderboards**: Paginated top balance queries with efficient ordering
- **UUID Resolution**: Name-to-UUID mapping for player operations
- **Schema Migration**: Automatic database schema migration on startup
- **CGO-Free**: All database drivers are pure Go implementations, no CGO required

## Domain Model
- **EconomyEntry**: Core domain struct with UUID, Name, Balance fields
- **Account**: Database model with GORM tags and automatic timestamps
- **Balance Field**: Renamed from "Money" to "Balance" for clarity
- **Configuration**: Multi-RDBMS configuration with database type, DSN, and default balance

## Configuration System
- **DBType**: Database type selection ("sqlite", "mysql", "postgres")
- **DBDSN**: Database connection string or path
- **DefaultBalance**: Initial balance for new users
- **EnableSetCmd**: Enable/disable the `/economy set` command (default: false)
- **String-Based Configuration**: Practical string-based settings for external library integration
- **Multi-RDBMS Support**: Configurable database backend selection
- **Command Control**: Selective command availability for enhanced security

## Security & Validation
- **Input Validation**: Amount validation (positive, minimum values)
- **Self-Transfer Prevention**: Cannot pay yourself
- **Command Access Control**: Set command disabled by default for security
- **Configurable Permissions**: Server admins control command availability
- **Timeout Handling**: All operations have 5-second timeout
- **Error Logging**: Internal errors logged with context
- **Transaction Safety**: GORM transactions ensure data consistency

## Information Retrieval
When detailed Dragonfly information is needed, use the DeepwikiMCP server:

- **Overview**: Use `mcp__deepwiki__ask_question` to get basic info about df-mc/dragonfly
- **Structure**: Use `mcp__deepwiki__read_wiki_structure` to check documentation structure
- **Command System**: Ask "How does the command system work?" for detailed specifications
- **Entity System**: Ask "How do entities work?" for implementation details
- **World Management**: Ask "How does world management work?" for world-related specifications

## Development Notes
- Commands use struct-based parameter definition (Dragonfly pattern)
- All database operations are context-aware with timeout handling
- Error messages use Minecraft color codes (§a for success, §c for error)
- Service layer provides clean separation between commands and database
- Demo server in `cmd/demo/main.go` shows complete integration example
- GORM provides ORM capabilities with automatic schema migration
- Domain-driven design with separate domain models and database models
- Configuration-driven approach with `config.Config` for multi-RDBMS customization and command control
- Clean architecture with clear separation of concerns
- Factory pattern implementation for database initialization
- Pure Go implementations for all database drivers (CGO-free)
- String-based configuration for practical external library integration

## Recent Changes
- **Command Access Control**: Added EnableSetCmd configuration for security control
- **Conditional Command Registration**: Set command only registered when explicitly enabled
- **Multi-RDBMS Support**: Implemented support for SQLite, MySQL, and PostgreSQL databases
- **Factory Pattern**: Added database factory for configurable RDBMS selection
- **Configuration Enhancement**: Extended config with DBType field for database selection
- **Field Renaming**: Changed DBPath to DBDSN for more accurate naming
- **String-Based Configuration**: Adopted practical string-based settings for external compatibility
- **Error Handling Reorganization**: Separated error types by layer responsibility
- **CGO-Free Implementation**: All database drivers use pure Go implementations
- **GORM Integration**: Migrated from raw SQL to GORM ORM for better data handling
- **Domain Model Refactoring**: Introduced clean domain models separate from database models

## Memories
- to memorize

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skuralll) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
