## architecture

> At the start of every response, say "APPLIED ARCHITECTURE RULE".

At the start of every response, say "APPLIED ARCHITECTURE RULE".

Code Architecture of /src

The /src directory implements a Model Context Protocol (MCP) server for OpenMetadata, providing programmatic access to OpenMetadata's REST API through a standardized interface. The architecture follows a modular, layered design that enables comprehensive data catalog and metadata management operations.

## Core Entry Points

**__main__.py**: Simple entry point that delegates to main()

**main.py**: Central orchestration module that:
- Configures CLI interface with Click for transport selection (stdio/sse)
- Dynamically loads and registers API modules based on user selection
- Maps API types to their respective function collections
- Initializes OpenMetadata client with environment-based authentication
- Manages server lifecycle with chosen transport protocol

**server.py**: MCP server definition and configuration providing:
- FastMCP server instance creation
- Transport abstraction layer (stdio/sse)
- Server lifecycle management

## Configuration Layer

**enums.py**: Defines APIType enum with OpenMetadata API categories:
- **Core Entities**: TABLE, DATABASE, SCHEMA, COLUMN
- **Data Assets**: DASHBOARD, CHART, PIPELINE, TOPIC
- **Data Quality**: DATAQUALITYTESTS, TESTCASES, METRICS, PROFILER
- **Governance**: CLASSIFICATION, TAG, GLOSSARY, POLICY
- **Lineage & Usage**: LINEAGE, USAGE, COST
- **System Management**: SERVICES, INGESTION, WEBHOOKS, BOTS

Environment variable management for OpenMetadata connection:
- `OPENMETADATA_HOST`: OpenMetadata server URL
- `OPENMETADATA_JWT_TOKEN`: JWT token for API authentication
- `OPENMETADATA_USERNAME`: Username for basic authentication
- `OPENMETADATA_PASSWORD`: Password for basic authentication

## OpenMetadata Integration Layer (/openmetadata/)

The architecture uses a consistent modular pattern across specialized API modules:

### Common Module Structure
Each module follows this standardized pattern:
- **API Client Setup**: Imports and configures specific OpenMetadata API client
- **Function Registry**: `get_all_functions()` returns list of (function, name, description) tuples
- **Async API Functions**: Transform OpenMetadata API calls into MCP-compatible responses
- **Tool Definitions**: MCP Tool objects with JSON schema validation

### Core Client Module

**openmetadata_client.py**: Centralized OpenMetadata REST API client configuration
- Authentication handling (JWT token or basic auth)
- HTTP session management with proper headers
- Error handling and response validation
- Base methods for CRUD operations on metadata entities
- Custom OpenMetadataError exception hierarchy

### API Group Modules

**table.py**: Comprehensive table management
- CRUD operations for table entities
- Field filtering and pagination support
- Fully qualified name (FQN) based lookups
- Soft and hard delete operations
- UI URL generation for web interface integration

**database.py**: Database container management
- Database lifecycle operations
- Service connection management
- Schema relationship handling

**schema.py**: Schema-level metadata management
- Schema CRUD operations
- Table relationship management
- Namespace organization

**dashboard.py**: Business intelligence dashboard operations
- Dashboard metadata management
- Chart relationship tracking
- Visualization configuration

**pipeline.py**: Data pipeline workflow management
- Pipeline definition and execution
- Task dependency tracking
- Workflow state management

**classification.py**: Data classification and sensitivity management
- Classification rule definitions
- Data sensitivity labeling
- Compliance tracking

**tag.py**: Tagging system operations
- Tag hierarchy management
- Entity tagging operations
- Tag-based search and filtering

**glossary.py**: Business glossary management
- Term definitions and relationships
- Business context documentation
- Terminology standardization

**lineage.py**: Data lineage tracking and visualization
- Upstream/downstream relationship mapping
- Impact analysis operations
- Lineage graph construction

**usage.py**: Data usage analytics and access patterns
- Usage metrics collection
- Access pattern analysis
- Performance monitoring

**services.py**: Data service configurations
- Service registration and management
- Connection testing and validation
- Service health monitoring

**ingestion.py**: Metadata ingestion pipeline management
- Ingestion workflow configuration
- Pipeline execution monitoring
- Error handling and retry logic

## Architecture Patterns

### Plugin Architecture
- Modular design allows selective API loading via CLI flags
- Each module is self-contained with clear interfaces
- Dynamic function registration enables extensibility
- Component-based structure supports incremental development

### Abstraction Layer
- Wraps OpenMetadata's REST API with async Python functions
- Standardizes response format using MCP TextContent types
- Handles parameter validation and API client configuration
- Provides consistent error handling across all operations

### Configuration Management
- Environment-based configuration for different OpenMetadata instances
- Multiple authentication methods (JWT tokens, basic auth)
- Host URL parsing and validation
- API versioning support
- Flexible authentication strategy selection

### Error Handling
- Custom OpenMetadataError exception hierarchy
- HTTP status code mapping to meaningful errors
- Graceful degradation when optional parameters are missing
- NotImplementedError handling for incomplete modules
- Comprehensive logging and debugging support

## Current Implementation Status

**Implemented Features**:
- Modular API group architecture
- Multiple authentication methods
- Transport flexibility (stdio/sse)
- Comprehensive error handling
- MCP standard compliance
- Dynamic module loading

**Architecture Benefits**:
- **Extensibility**: Easy addition of new OpenMetadata entity types
- **Maintainability**: Clear separation of concerns and modular design
- **Testability**: Isolated components with clear interfaces
- **Performance**: Efficient HTTP session management and connection pooling
- **Standards Compliance**: Full MCP protocol implementation
- **Selective Deployment**: Enable only required API groups for specific use cases

## Future Architecture Enhancements

The modular design supports natural expansion to cover OpenMetadata's full API surface:

1. **Complete API Coverage**: Implement all 12+ planned API group modules
2. **Batch Operations**: Support for bulk operations and batch processing
3. **Real-time Updates**: WebSocket or SSE support for live metadata updates
4. **Advanced Search**: Integration with OpenMetadata's search and discovery capabilities
5. **Workflow Integration**: Support for complex metadata workflows and approvals
6. **Performance Optimization**: Caching layers and connection pooling
7. **Event-Driven Architecture**: WebHook integration for real-time notifications

## Module Implementation Priority

**Phase 1 (Core Operations)**:
- table.py, database.py, schema.py

**Phase 2 (Data Assets)**:
- dashboard.py, pipeline.py, services.py

**Phase 3 (Governance & Quality)**:
- classification.py, tag.py, glossary.py

**Phase 4 (Analytics & Monitoring)**:
- lineage.py, usage.py, ingestion.py

This architecture provides a clean, extensible interface to OpenMetadata's comprehensive data catalog capabilities while maintaining separation of concerns and enabling selective functionality deployment based on user needs.

---
> Source: [yangkyeongmo/mcp-server-openmetadata](https://github.com/yangkyeongmo/mcp-server-openmetadata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
