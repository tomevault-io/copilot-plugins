## pokearchive

> * **Layers** - standardized levels of abstraction


# Server Architecture Guide

## Overview

* **Layers** - standardized levels of abstraction
* **Slices** - feature-based divisions within layers
* **Segments** - technical purpose divisions within slices

```
┌─────────────┬────────────┬────────────┐
│   Layers    │   Slices   │  Segments  │
├─────────────┼────────────┼────────────┤
│   routes    │      -     │      +     │
│   modules   │      +     │      +     │
│   widget    │      +     │      +     │
│   features  │      +     │      +     │
│   entities  │      +     │      +     │
│   shared    │      -     │      +     │
└─────────────┴────────────┴────────────┘
```


## Complete Project Structure

```
src/
├── server.ts                                       # Main entry point - Fastify app initialization
├── payload.config.ts                               # Payload CMS configuration
├── payload-types.ts                                # Payload CMS generated types
├── app/
│   ├── (payload)/                                  # Layer - Payload CMS admin panel
│   │   ├── [[...segments]]/                        # Catch-all route for Payload admin
│   │   ├── api/                                    # Payload API routes
│   │   ├── migrations/                             # Database migrations
│   │   ├── seed/                                   # Database seed data
│   │   ├── layout.tsx                              # Admin layout
│   │   ├── custom.scss                             # Admin custom styles
│   │   └── importMap.js                            # Payload import map
│   ├── routes/                                     # Layer - API routing
│   │   ├── index.ts                                # Main router export
│   │   └── server.routes.ts                        # Route definitions and module registration
│   ├── modules/                                    # Layer - Main business logic
│   │   ├── module-name/                            # Slice
│   │   │   ├── module-name.module.ts               # Route registration and validation
│   │   │   ├── module-name.service.ts              # Business logic
│   │   │   └── index.ts
│   │   └── ...
│   ├── widget/                                     # Layer - Complex reusable functionality
│   │   └── widget-name/                            # Slice
│   │       ├── widget-name.service.ts              # Widget logic
│   │       ├── widget-name.interface.ts            # Type definitions (Optional)
│   │       ├── widget-name.constant.ts             # Constants (Optional)
│   │       └── index.ts
│   ├── features/                                   # Layer - Reusable implementations
│   │   ├── feature-name/                           # Slice
│   │   │   ├── feature-name.service.ts             # Feature logic
│   │   │   ├── feature-name.interface.ts           # Type definitions (Optional)
│   │   │   ├── feature-name.constant.ts            # Constants (Optional)
│   │   │   └── index.ts
│   │   └── ...
│   ├── entities/                                   # Layer - Business entities
│   │   ├── collections/                            # Slice - Payload CMS collections
│   │   │   ├── entity-name.collection.ts           # Collection definition
│   │   │   ├── ...
│   │   │   └── index.ts
│   │   ├── dto/                                    # Slice - Data transfer objects
│   │   │   ├── entity-name.dto.ts                  # Zod validation schemas
│   │   │   ├── ...
│   │   │   └── index.ts
│   │   └── index.ts
│   └── shared/                                     # Layer - Reusable code
│       ├── components/                             # Segment - Shared components
│       ├── constant/                               # Segment - Global constants
│       │   ├── constant-name.constant.ts
│       │   └── index.ts
│       ├── decorators/                             # Segment - Payload CMS decorators/hooks
│       │   └── hook-name.hook.ts
│       ├── hooks/                                  # Segment - Custom hooks
│       │   ├── hook-name.hook.ts
│       │   └── index.ts
│       ├── interface/                              # Segment - Global interface definitions
│       │   ├── interface-name.interface.ts
│       │   └── index.ts
│       ├── middleware/                             # Segment - Middlewares
│       ├── service/                                # Segment - Shared services
│       │   ├── service-name.service.ts
│       │   └── index.ts
│       └── util/                                   # Segment - Utility functions
│           ├── util-name.util.ts
│           └── index.ts
├── config/                                         # Application configuration
│   ├── env.config.ts                               # Environment variables validation
│   ├── server.config.ts                            # Server configuration (cors, rate-limit, etc.)
│   ├── locale.config.ts                            # Locale configuration
│   ├── swagger.config.ts                           # Swagger documentation configuration
│   └── index.ts
└── pkg/                                            # External packages/utilities
    ├── auth/                                       # Authentication (plugin, service, constants)
    ├── cache/                                      # Cache client (Redis)
    ├── customer-io/                                # Customer.io integration
    ├── middleware/                                 # Global middleware
    ├── mixpanel/                                   # Mixpanel analytics
    ├── payload/                                    # Payload CMS utilities
    │   ├── db/                                     # Database service
    │   ├── fields/                                 # Custom Payload fields
    │   └── plugins/                                # Custom Payload plugins
    └── storage/                                    # S3 storage client
```


## Layer Descriptions

### 1. Routes Layer
**Purpose**: API routing and module registration
- Define API route prefixes
- Register modules as Fastify plugins
- Health check endpoint
- Catch-all 404 handler

**Files**:
- `server.routes.ts` - Main route definitions and module registration

### 2. Modules Layer
**Purpose**: Core business logic
- Implement business rules
- Handle HTTP requests/responses
- Request/Response validation with Zod
- Orchestrate operations

**Files**:
- `*.module.ts` - Route registration, validation schemas, request handlers
- `*.service.ts` - Business logic implementation

### 3. Widget Layer
**Purpose**: Complex reusable functionality
- Multi-step operations
- Composite functionality
- Reusable across multiple modules

**Files**:
- `*.service.ts` - Widget logic
- `*.interface.ts` - Widget-specific types
- `*.constant.ts` - Widget-specific constants

### 4. Features Layer
**Purpose**: Reusable implementations (Payload CMS blocks, components)
- Payload CMS block definitions
- Custom admin components
- Reusable across collections

**Files**:
- `*.block.ts` - Payload CMS block definitions
- `*.component.tsx` - Custom admin components

### 5. Entities Layer
**Purpose**: Business entities and data structures
- Payload CMS collection definitions
- Zod validation schemas (DTO)
- Entity types

**Files**:
- `collections/*.collection.ts` - Payload CMS collection definitions
- `dto/*.dto.ts` - Zod validation schemas (Data Transfer Objects)

### 6. Shared Layer
**Purpose**: Common utilities and helpers
- Utility functions
- Shared services
- Global constants and interfaces
- Custom hooks and decorators

**Files**:
- `service/*.service.ts` - Shared services
- `util/*.util.ts` - Helper functions
- `constant/*.constant.ts` - Global constants
- `interface/*.interface.ts` - Global types
- `hooks/*.hook.ts` - Custom hooks
- `decorators/*.hook.ts` - Payload CMS hooks/decorators


## Import Rules

Layers can only import from lower layers in the hierarchy:

```typescript
// ✅ Allowed imports
// routes -> modules, widget, features, entities, shared, config, pkg
// modules -> widget, features, entities, shared, config, pkg
// widget -> features, entities, shared, config, pkg
// features -> entities, shared, config, pkg
// entities -> shared, config, pkg
// shared -> config, pkg
// config -> (no other layers, only pkg)

// ❌ Forbidden imports
// shared -> entities, features, widget, modules, routes
// entities -> features, widget, modules, routes
// features -> widget, modules, routes
// widget -> modules, routes
// modules -> routes
// config -> any app layers
```


## Naming Conventions

### Files
- **Modules**: `module-name.module.ts`
- **Services**: `service-name.service.ts`
- **Routes**: `server.routes.ts`
- **Collections**: `entity-name.collection.ts`
- **DTOs**: `entity-name.dto.ts`
- **Blocks**: `block-name.block.ts`
- **Components**: `component-name.component.tsx`
- **Middlewares**: `middleware-name.middleware.ts`
- **Utilities**: `util-name.util.ts`
- **Constants**: `constant-name.constant.ts`
- **Interfaces**: `interface-name.interface.ts`
- **Hooks**: `hook-name.hook.ts`
- **Plugins**: `plugin-name.plugin.ts`
- **Clients**: `client-name.client.ts`

### Directories
- Use **kebab-case**: `customer-progress`, `customer-io`
- Singular for slices: `widget`, `auth`
- Plural for segment collections: `collections`, `hooks`


## Project Structure Rules

1. **Each layer is independent** - Lower layers don't know about upper layers
2. **Index files** - Export public API of each slice
3. **No circular dependencies** - Use dependency injection if needed
4. **Keep routes thin** - Business logic belongs in services
5. **Type safety first** - Use TypeScript strictly
6. **Single responsibility** - Each file has one clear purpose


## Testing Structure

```
tests/
├── int/
│   └── api.int.spec.ts
└── e2e/
    └── frontend.e2e.spec.ts
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vitalina-kostenko-js) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
