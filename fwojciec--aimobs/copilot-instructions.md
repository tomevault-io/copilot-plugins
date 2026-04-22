## aimobs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIMobs is a Minecraft Fabric mod that creates AI-powered voice-controlled mob entities. The system enables players to communicate with in-game mobs using natural language, with mobs responding through AI-generated speech and actions. The mod integrates with external APIs (OpenAI Whisper, GPT-4, 11 Labs) via WebSocket connections.

## Build Commands

- **Build the mod**: `./gradlew build`
- **Run unit tests**: `./gradlew test`
- **Run integration tests**: `./gradlew integrationTest`
- **Launch Minecraft client**: `./gradlew runClient`
- **Launch test server**: `./gradlew runServer`
- **Generate sources**: `./gradlew genSources`
- **Clean build**: `./gradlew clean`

## Development Environment

- **Java Version**: Java 21 (required for Minecraft 1.20.4)
- **Minecraft Version**: 1.20.4
- **Fabric Loader**: 0.15.3
- **Fabric API**: 0.92.0
- **Build System**: Gradle with Fabric Loom 1.4.6

## Architecture Overview

The project follows **Ports and Adapters Architecture** with **Standard Package Layout** principles:

### Core Layers
- **Root Package (`entity.ai/`)**: Service interfaces and contracts
- **Core Package (`core/`)**: Domain primitives (AICommand, AIState, EntityActions)
- **Application Package (`application/`)**: Business logic implementations
- **Infrastructure Package (`infrastructure/`)**: Minecraft-specific adapters

### Key Services
- **ServiceFactory**: Composition root for dependency injection
- **CommandProcessorService**: Executes AI commands on entities
- **WebSocketService**: Handles real-time communication with external AI services
- **MovementService**: Coordinates entity movement and pathfinding
- **InteractionService**: Handles entity interactions and inventory management

### Testing Architecture
- **Unit Tests**: Fast tests using service interfaces
- **Integration Tests**: Test service coordination with `@Tag("integration")`
- **Test Doubles**: Use fakes for business logic, mocks only for external APIs
- **Seam Pattern**: All services injectable via interfaces for testability

## Important Design Principles

1. **Interface-First Development**: Always create interfaces before implementations
2. **Unidirectional Dependencies**: Infrastructure → Application → Core
3. **No Peer Dependencies**: Application and Infrastructure cannot depend on each other
4. **Composition Root**: All object construction happens in ServiceFactory
5. **Testability**: Every component must be testable in isolation

## WebSocket Integration

The mod connects to external services via WebSocket on port 8080 (configurable via system property `aimobs.websocket.url`). Network communication uses structured JSON messages for command processing and AI responses.

## Common Development Tasks

- **Add new AI command**: Implement `AICommand` interface in `core/` package
- **Create new service**: Add interface to root package, implementation to `application/`
- **Extend entity behavior**: Add methods to `EntityActions` interface
- **Add test**: Use appropriate test category tags and follow seam patterns

## Project Structure Notes

- Entity AI logic is separated by architectural layer
- WebSocket networking is in `network/` package with same layered structure  
- Tests mirror main source structure with comprehensive unit and integration coverage
- External API integrations are abstracted through service interfaces

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fwojciec) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
