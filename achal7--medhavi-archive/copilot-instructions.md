## medhavi-archive

> Comprehensive project structure documentation for Medhāvī supply chain platform with detailed layer explanations

# Medhāvī Project Structure & Architecture Layers

## Overview

Medhāvī is a supply chain optimization platform built with F# using Domain-Driven Design (DDD), Event-Driven Architecture (EDA), and Clean Architecture principles. The system follows CQRS/ES patterns with event sourcing.

## Architecture Layers

### 1. Domain Layer (`Medhavi.Domain`)
**Purpose**: Pure business domain contracts and rules
- **Contents**: Aggregate types, commands, events, domain signatures only
- **Rule**: NO implementations, NO business logic, NO external dependencies
- **Examples**:
  - `UnitOfMeasure.fs` - Unit aggregate contracts
  - `Supplier.fs` - Supplier aggregate contracts
  - `SupplyOrder.fs` - Supply order aggregate contracts

### 2. Data Transfer Objects (`Medhavi.DTO`)
**Purpose**: External system integration contracts
- **Contents**: DTO types, event type constants
- **Rule**: Simple data structures for serialization/deserialization
- **Examples**:
  - `UnitConversionDto` - For unit conversion events
  - `ProductDto` - For product integration events

### 3. Projections Layer (`Medhavi.Projections`)
**Purpose**: Read models for optimized querying
- **Contents**: Event handlers, query services, in-memory state management
- **Rule**: Pure functions for state evolution, no business logic
- **Examples**:
  - `UnitOfMeasure.fs` - Unit read models and queries
  - `Supplier.fs` - Supplier read models and queries

### 4. Foundation Layer (`Medhavi.Foundation`)
**Purpose**: Infrastructure for event-driven architecture
- **Contents**: Event store, actors, serialization, logging, configuration
- **Rule**: Framework code, no business logic
- **Examples**:
  - `Aggregate.fs` - Aggregate pattern implementation
  - `EnvelopeStore.fs` - Event persistence
  - `BrokerActor.fs` - Integration messaging

### 5. Nexus Application (`Medhavi.Nexus`)
**Purpose**: Master data management and control tower
- **Contents**: Aggregate lifecycle operations, REST APIs, ACL layers
- **Rule**: Business logic for CRUD operations on master data
- **Examples**:
  - `Features/UnitOfMeasure/` - Unit lifecycle management
  - `Features/Supplier/` - Supplier lifecycle management

### 6. Planning Application (`Medhavi.Planning`)
**Purpose**: Advanced planning and optimization algorithms
- **Contents**: Domain services, planning algorithms, optimization logic
- **Rule**: Complex business logic for planning operations
- **Examples**:
  - `Features/UnitOfMeasure/DomainServices.fs` - Unit conversion services
  - `Optimization/Solver.fs` - Planning algorithms

## Project Structure Rules

### Source Code Organization (`src/`)

#### Libraries (`src/Libraries/`)

##### `Medhavi.Common/`
- **Purpose**: Shared utilities and functional patterns
- **Contents**: Result types, Task patterns, validation helpers, serialization
- **Rule**: No business logic, pure utilities only

##### `Medhavi.Domain/`
- **Purpose**: Core business domain models
- **Structure**:
  ```
  Common/           # Shared domain types
  Material/         # Material planning domain
  Capacity/         # Capacity management domain
  Resources/        # Resource management domain
  Promise/          # Order promising domain
  Calendar/         # Calendar and scheduling domain
  ```
- **Rules**:
  - Only type definitions, commands, events, and signatures
  - No `let` bindings or implementations
  - No external dependencies
  - Each aggregate in separate file
  - Strong-typed IDs in `Common/Ids.fs`

##### `Medhavi.DTO/`
- **Purpose**: External system integration
- **Contents**: DTO types and event constants
- **Rules**:
  - Simple record types only
  - Event type literals for routing
  - No business logic

##### `Medhavi.Foundation/`
- **Purpose**: Event-driven infrastructure
- **Structure**:
  ```
  Aggregate/        # Aggregate pattern implementation
  Integration/      # External system integration
  Projections/      # Projection infrastructure
  Saga/            # Saga pattern implementation
  Stores/          # Event store implementations
  Observability/   # Telemetry and monitoring
  ```
- **Rules**:
  - Framework code only
  - No business domain logic
  - Reusable across applications

##### `Medhavi.Projections/`
- **Purpose**: Read model implementations
- **Contents**: Event handlers and query services
- **Rules**:
  - Pure functions for state evolution
  - Query services return immutable data
  - Shared across Nexus and Planning applications

#### Server Applications (`src/Server/`)

##### `Medhavi.Nexus/`
- **Purpose**: AI-powered control tower for master data
- **Structure**:
  ```
  Features/         # Feature modules
    UnitOfMeasure/  # Unit lifecycle management
    Supplier/       # Supplier lifecycle management
    Product/        # Product lifecycle management
    ...
  Infrastructure/   # Application infrastructure
  Services/         # Application services
  ```
- **Feature Module Structure**:
  ```
  FeatureName/
    ACL.fs           # Anti-corruption layer
    Application.fs   # Business logic & workflow
    Controller.fs    # REST API endpoints
    Domain.fs        # Domain validations & state transitions
    FeatureConfig.fs # CQRS wiring & system integration
    Transformation.fs # Event serialization
  ```
- **Rules**:
  - One feature module per aggregate
  - ACL validates external data before domain processing
  - Controllers expose CRUD operations
  - Business invariants enforced in Application layer

##### `Medhavi.Planning/`
- **Purpose**: Tactical planning and optimization engine
- **Structure**:
  ```
  Features/         # Planning features
    UnitOfMeasure/  # Domain services for units
    Supplier/       # Domain services for suppliers
    OrderAcceptance/# Order promising logic
    ...
  Optimization/     # Planning algorithms
  Promise/          # Promise/commitment logic
  Capacity/         # Capacity planning
  ```
- **Rules**:
  - Domain services consume projections for planning operations
  - Complex algorithms in Optimization/
  - Planning logic separated from master data management

## Development Rules

### Code Organization
- **One aggregate per file** in domain layer
- **Feature modules** group related functionality
- **ACL validates** all external input before domain processing
- **Domain services** in planning layer for complex operations
- **Shared projections** across applications

### Dependencies
- **Domain layer**: No external dependencies
- **Projections**: Depend on Domain and Foundation
- **Applications**: Depend on all layers
- **Foundation**: Independent infrastructure layer

### Naming Conventions
- **Aggregates**: PascalCase, descriptive names
- **Commands**: End with `Cmd` (e.g., `CreateSupplierCmd`)
- **Events**: End with `Evt` (e.g., `SupplierCreatedEvt`)
- **Services**: End with `Service` (e.g., `UnitConversionService`)

### File Organization
- **Domain types**: `*.fs` files in `Medhavi.Domain/`
- **DTO types**: `Library.fs` in `Medhavi.DTO/`
- **Projections**: One file per aggregate in `Medhavi.Projections/`
- **Features**: One directory per feature in application layers

## Architecture Principles

### Clean Architecture
- **Dependency Rule**: Inner layers don't depend on outer layers
- **Domain at Center**: Business rules independent of frameworks
- **Boundaries**: Clear separation between layers

### DDD Patterns
- **Aggregates**: Transaction boundaries around related entities
- **Value Objects**: Immutable objects representing concepts
- **Domain Services**: Operations that don't belong to single aggregate
- **Anti-Corruption Layer**: Translation between bounded contexts

### Event-Driven Architecture
- **Event Sourcing**: State changes as immutable events
- **CQRS**: Separate read/write models
- **Projections**: Event handlers build optimized read models
- **Integration Events**: Cross-system communication

This structure ensures maintainable, testable, and scalable supply chain optimization platform.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/achal7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
