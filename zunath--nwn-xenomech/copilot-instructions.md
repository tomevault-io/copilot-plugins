## nwn-xenomech

> This is a C# server-side solution for the Neverwinter Nights: Enhanced Edition Xenomech module. It replaces traditional NWScript with C# code using the NWNX_DotNet plugin and Anvil framework.

# Xenomech - Neverwinter Nights Server Solution

## Project Overview
This is a C# server-side solution for the Neverwinter Nights: Enhanced Edition Xenomech module. It replaces traditional NWScript with C# code using the NWNX_DotNet plugin and Anvil framework.

## External Dependencies

### NWNX:EE Unified
- **Repository**: https://github.com/nwnxee/unified
- **Purpose**: Core NWNX framework that provides plugins for extending NWN server functionality
- **Key Plugin**: NWNX_DotNet - Enables C#/.NET integration with NWN server
- **Configuration**: Environment variables control plugin activation (see `XM.App.Runner/Docker/nwserver.env`)

### Anvil Framework
- **Repository**: https://github.com/nwn-dotnet/Anvil
- **Purpose**: .NET framework for NWN server development
- **Usage**: All plugins in this solution are built as Anvil plugins
- **Integration**: Provides the base classes and interfaces for NWN server plugins

## External Documentation

### Design Documents
- **XM Design Bible**: https://docs.google.com/spreadsheets/d/1CnS5sk6c9cjlRETEuAkkyI5P1gPSskJWct7Qemx-KVs/edit?gid=776582455#gid=776582455
  - Contains comprehensive game design specifications including attributes, stats, weapon skills, armor, jobs, and other game mechanics
  - Reference for implementing character progression, combat systems, and item mechanics
  - Includes XP charts, damage calculations, and spell damage formulas

- **Additional Design Document**: https://docs.google.com/document/d/1D24AT6qYXE9DDVpsuVY9D4jW-2cNlvTWPrx38P1topI/edit?tab=t.0#heading=h.5mpdqkyy0jba
  - Contains additional design specifications and implementation details
  - Reference for game mechanics and system design decisions

## Architecture

### Solution Structure
- **XM.App.Runner**: Main application that orchestrates Docker containers and NWN server
- **XM.Shared.API**: Core API layer with NWScript bindings and engine structures
- **XM.Shared.Core**: Shared core functionality (chat commands, authorization, etc.)
- **XM.Shared.AI**: AI system with behavior trees and targeting
- **XM.Shared.Inventory**: Inventory management system
- **XM.Shared.Progression**: Character progression and leveling
- **XM.Shared.UI**: User interface components

### Plugin Architecture
Each plugin follows the Anvil framework pattern:
- **XM.Plugin.Administration**: Server administration and moderation
- **XM.Plugin.Area**: Area management and persistent locations
- **XM.Plugin.Chat**: Chat system and communication
- **XM.Plugin.Combat**: Combat mechanics and status effects
- **XM.Plugin.Item**: Item management, appearance editor, market system
- **XM.Plugin.Migration**: Player and server migration utilities
- **XM.Plugin.Quest**: Quest system with objectives and rewards
- **XM.Plugin.Spawn**: Creature spawning and management

### Content Structure
- **Content/**: Game assets (models, textures, tilesets, etc.)
- **Module/**: NWN module files (areas, dialogues, items, etc.)
- **Tools/**: Build tools and utilities

## Development Environment

### Prerequisites
- Git
- Docker Desktop
- Visual Studio 2022
- Neverwinter Nights: Enhanced Edition

### Build Configuration
- **Target Framework**: .NET 8.0
- **Runtime**: linux-x64 (for Docker deployment)
- **Platform**: x64
- **Language Version**: default

### Key Build Properties
- `EnableDynamicLoading: true` - Required for plugin loading
- `CopyLocalLockFileAssemblies: true` - Ensures all dependencies are copied
- `AppendTargetFrameworkToOutputPath: false` - Simplified output structure

## Docker Integration

### Container Setup
- Uses Docker Compose for orchestration
- NWN server runs in Linux container with NWNX plugins
- Redis for caching and session management
- InfluxDB for metrics collection

### Environment Variables
Key NWNX configuration in `nwserver.env`:
- `NWNX_DOTNET_SKIP=n` - Enables .NET plugin
- `NWNX_CORE_LOG_LEVEL=6` - Debug logging
- Various plugin enable/disable flags

## Code Patterns

### Plugin Development
```csharp
// All plugins inherit from Anvil base classes
// Use dependency injection for services
// Subscribe to NWN events for game integration
```

### Event Handling
```csharp
// Subscribe to NWNX events for game integration
// Use async/await for non-blocking operations
// Implement proper error handling and logging
```

### Database Integration
```csharp
// Use DBService for entity persistence
// Implement proper migration strategies
// Handle connection pooling and transactions
```

## Key Features

### Game Systems
- **Combat**: Custom combat mechanics with status effects
- **Quest System**: Dynamic quests with objectives and rewards
- **Item System**: Advanced item management with appearance editor
- **AI System**: Behavior tree-based AI with targeting
- **Chat System**: Roleplay features and communication tools
- **Administration**: Moderation tools and server management

### Technical Features
- **Hot Reload**: Development-time plugin reloading
- **Metrics**: Performance monitoring with InfluxDB
- **Logging**: Comprehensive logging system
- **Docker**: Containerized deployment
- **Database**: Persistent data storage

## Development Workflow

1. **Setup**: Clone repo, install prerequisites
2. **Build**: Use Visual Studio or command line
3. **Run**: Execute `XM.App.Runner` to start Docker containers
4. **Debug**: Attach debugger to running container
5. **Deploy**: Use Docker Compose for production

## Common Patterns

### Service Registration
```csharp
// Register services in dependency injection container
// Use scoped/transient/singleton appropriately
// Implement interfaces for testability
```

### Event Subscription
```csharp
// Subscribe to NWN events in plugin constructors
// Use proper event handling patterns
// Implement cleanup in disposal
```

### Configuration
```csharp
// Use appsettings.json for configuration
// Environment variables for Docker deployment
// Feature flags for runtime configuration
```

## Testing Strategy
- **Unit Tests**: Test individual components
- **Integration Tests**: Test plugin interactions
- **End-to-End**: Test complete game scenarios

## Performance Considerations
- **Async Operations**: Use async/await for I/O operations
- **Caching**: Implement caching for frequently accessed data
- **Connection Pooling**: Reuse database connections
- **Memory Management**: Proper disposal of resources

## Security
- **Input Validation**: Validate all user inputs
- **Authorization**: Implement proper access controls
- **Data Sanitization**: Sanitize data before persistence
- **Logging**: Audit trail for administrative actions

## Troubleshooting
- **Docker Issues**: Check container logs and environment variables
- **Plugin Loading**: Verify DLL dependencies and paths
- **Database**: Check connection strings and migrations
- **Performance**: Monitor metrics and logs for bottlenecks 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zunath) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
