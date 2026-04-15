## netstr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Run
- `dotnet run --project .\src\Netstr\Netstr.csproj` - Run the main application
- `dotnet build` - Build the solution
- `dotnet build --configuration Release` - Build for release

### Testing
- `dotnet test` - Run all tests
- `dotnet test --filter "DisplayName~<scenario_name>"` - Run specific SpecFlow scenario
- `dotnet test test/Netstr.Tests/Netstr.Tests.csproj` - Run tests for specific project

### Database
- `dotnet ef migrations add <name> --project src/Netstr` - Add new EF migration
- `dotnet ef database update --project src/Netstr` - Apply migrations

### Docker
- `docker build -t netstr .` - Build Docker image
- `docker compose up` - Run with Docker Compose (includes PostgreSQL)

## Architecture Overview

Netstr is a modern Nostr relay written in C# using ASP.NET Core, targeting .NET 9.0. It implements multiple NIPs (Nostr Implementation Possibilities) for the decentralized nostr protocol.

### Core Components

**WebSocket Message Processing Pipeline:**
1. `WebSocketAdapter` - Handles WebSocket connections and message routing
2. `MessageDispatcher` - Routes incoming messages to appropriate handlers based on message type
3. `EventDispatcher` - Routes EVENT messages to specific event handlers based on event kind

**Message Handlers** (in `src/Netstr/Messaging/MessageHandlers/`):
- `SubscribeMessageHandler` - Handles REQ (subscription) messages
- `UnsubscribeMessageHandler` - Handles CLOSE messages
- `AuthMessageHandler` - Handles AUTH messages for NIP-42
- `CountMessageHandler` - Handles COUNT messages for NIP-45
- `NegentropyMessageHandler` / `NegentropyOpenHandler` / `NegentropyCloseHandler` - Handle negentropy sync (NIP-77)

**Event Handlers** (in `src/Netstr/Messaging/Events/Handlers/`):
- `RegularEventHandler` - Standard events (kind 1, etc.)
- `DeleteEventHandler` - Event deletion (NIP-09)
- `EphemeralEventHandler` - Ephemeral events (kinds 20000-29999)
- `ReplaceableEventHandler` - Replaceable events (kinds 10000-19999)
- `AddressableEventHandler` - Addressable events (kinds 30000-39999)
- `ZapEventHandler` - Zap events (NIP-57)
- `RelayListEventHandler` - Relay list events
- `VanishEventHandler` - Vanish requests (NIP-62)

**Data Layer:**
- Uses Entity Framework Core with PostgreSQL
- `NetstrDbContext` - Main database context
- `EventEntity`, `TagEntity`, `RelayConfigEntity` - Core data models
- Migrations in `src/Netstr/Data/Migrations/`

**Configuration:**
- `appsettings.json` / `appsettings.Development.json` - Main configuration
- Options pattern used throughout (`AuthOptions`, `LimitsOptions`, `WhitelistOptions`, etc.)
- `ConnectionOptions` for WebSocket configuration

### Key Features
- **Whitelist Management:** Public key-based access control for publishing/subscribing
- **Event Validation:** Multi-layer validation including signatures, PoW, timestamps, and custom validators
- **Subscription Management:** Efficient filtering and real-time event delivery
- **Negentropy Sync:** Advanced synchronization protocol for relay-to-relay communication
- **Rate Limiting:** Built-in limits for events, subscriptions, and payload sizes
- **Authentication:** NIP-42 auth support with challenge-response

### Testing Strategy
- Uses SpecFlow with Gherkin scenarios for behavior-driven testing
- Test scenarios written in plain English in `.feature` files
- Step definitions in `test/Netstr.Tests/NIPs/Steps/`
- Each NIP has dedicated test scenarios
- Uses xUnit as the underlying test framework
- Test data in `test/Netstr.Tests/Resources/Events.json`

### Message Flow
1. WebSocket connection established → `WebSocketAdapter.StartAsync()`
2. Raw message received → `MessageDispatcher.DispatchMessageAsync()`
3. Message parsed and routed to appropriate `IMessageHandler`
4. If EVENT message → `EventDispatcher.DispatchEventAsync()` → specific `IEventHandler`
5. Event validation through multiple `IEventValidator` implementations
6. Event stored to database or processed ephemerally
7. Event distributed to matching subscriptions

### Development Notes
- Uses dependency injection throughout
- Logging via Serilog
- Database migrations handled automatically on startup
- WebSocket adapters managed in collections for broadcast scenarios
- Custom JSON serialization for Nostr protocol compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmmanuelAlmonte)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EmmanuelAlmonte)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
