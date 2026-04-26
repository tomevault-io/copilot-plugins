## graphql-auth

> Main cursor rules for GraphQL Auth project. Read this first!

# Main Project Guide

This file provides a comprehensive overview of the project, including architecture, commands, and key development patterns.

## Quick Command Reference

```bash
# Development
bun run dev                             # Start dev server (port 4000)
bun run test --run                      # Run all tests once (no watch)
bun run test                            # Run tests in watch mode
bun test test/auth.test.ts              # Run specific test file
bun run test -t "test name"             # Run tests matching pattern
bun run test:ui                         # Run tests with UI
bun run test:coverage                   # Run tests with coverage

# Database
bunx prisma migrate dev --name feature  # Create migration
bun run generate                        # Generate all types (Prisma + GraphQL)
bun run db:reset                        # Reset database with seed data
bunx prisma studio                      # Open database GUI

# Build & Production
bun run build                          # Build for production (builds src/main.ts)
bun run start                          # Start production server (runs dist/app/server.js)
bun run clean                          # Clean build directory

# Code Quality
bunx tsc --noEmit                      # Type check all files
bunx prettier --write .                # Format code

# GraphQL Schema
bun run gen:schema                      # Generate GraphQL schema file
bunx gql.tada generate-output           # Generate GraphQL type definitions

# Environment & Debugging
bun run env:verify                      # Verify environment setup
bun run graphql:examples                # Run GraphQL query examples
```

## Architecture Overview

### Tech Stack

- **Runtime**: Bun (fast JavaScript/TypeScript runtime)
- **GraphQL Server**: Apollo Server 4 with H3 HTTP framework
- **Schema Builder**: Pothos with advanced plugin integration:
  - **Prisma Plugin**: Direct access pattern (NOT in context) for better TypeScript performance
  - **Relay Plugin**: Global IDs, connections with metadata, cursor pagination
  - **Errors Plugin**: Union result types for comprehensive error handling
  - **Scope Auth Plugin**: Dynamic authorization with 14+ scope types
  - **DataLoader Plugin**: Automatic batch loading for N+1 prevention
  - **Validation Plugin**: Zod integration with async refinements
- **Database**: Prisma ORM with SQLite (dev.db) - accessed directly, not through context
- **Authentication**: JWT tokens with bcryptjs + refresh token rotation
- **Authorization**: GraphQL Shield middleware with Pothos Scope Auth
- **Type Safety**: GraphQL Tada for compile-time GraphQL typing
- **Testing**: Vitest with comprehensive test utilities
- **Validation**: Zod schema validation with custom async refinements
- **Rate Limiting**: rate-limiter-flexible with configurable presets
- **Dependency Injection**: TSyringe for service management

### Architecture: Modular Direct Resolvers

The project uses a **modular architecture** with direct Pothos resolvers organized by feature:

```
src/
├── app/                              # Application entry and configuration
│   ├── [server.ts](mdc:src/app/server.ts)                     # Main server setup (entry point: bun run dev)
│   ├── config/                       # Configuration
│   │   ├── [container.ts](mdc:src/app/config/container.ts)              # TSyringe DI container setup
│   │   ├── [environment.ts](mdc:src/app/config/environment.ts)            # Environment variable handling
│   │   └── [database.ts](mdc:src/app/config/database.ts)               # Database configuration
│   └── middleware/                   # Application-level middleware
│       ├── [cors.ts](mdc:src/app/middleware/cors.ts)                   # CORS configuration
│       ├── [rate-limiting.ts](mdc:src/app/middleware/rate-limiting.ts)          # Rate limiter setup
│       └── [logging.ts](mdc:src/app/middleware/logging.ts)                # Request/response logging
│
├── modules/                          # Feature modules (domain-driven)
│   ├── auth/                         # Authentication module
│   │   ├── [auth.schema.ts](mdc:src/modules/auth/auth.schema.ts)            # GraphQL schema definitions
│   │   ├── [auth.permissions.ts](mdc:src/modules/auth/auth.permissions.ts)       # Authorization rules
│   │   ├── [auth.validation.ts](mdc:src/modules/auth/auth.validation.ts)        # Input validation schemas
│   │   ├── resolvers/                # GraphQL resolvers
│   │   │   ├── [auth.resolver.ts](mdc:src/modules/auth/resolvers/auth.resolver.ts)      # Basic auth (signup/login/me)
│   │   │   └── [auth-tokens.resolver.ts](mdc:src/modules/auth/resolvers/auth-tokens.resolver.ts) # Refresh token operations
│   │   ├── services/                 # Business logic services
│   │   │   ├── [password.service.ts](mdc:src/modules/auth/services/password.service.ts)   # Password hashing (bcryptjs)
│   │   │   └── [token.service.ts](mdc:src/modules/auth/services/token.service.ts)      # JWT token management
│   │   └── types/                    # Module-specific types
│   ├── posts/                        # Posts module
│   │   ├── [posts.schema.ts](mdc:src/modules/posts/posts.schema.ts)           # Post type definitions
│   │   ├── [posts.permissions.ts](mdc:src/modules/posts/posts.permissions.ts)      # Post authorization rules
│   │   ├── [posts.validation.ts](mdc:src/modules/posts/posts.validation.ts)       # Post input validation
│   │   ├── [posts.service.ts](mdc:src/modules/posts/posts.service.ts)          # Post business logic
│   │   └── resolvers/                # Post CRUD resolvers
│   ├── users/                        # Users module
│   │   ├── [users.schema.ts](mdc:src/modules/users/users.schema.ts)           # User type definitions
│   │   ├── [users.permissions.ts](mdc:src/modules/users/users.permissions.ts)      # User authorization rules
│   │   ├── [users.validation.ts](mdc:src/modules/users/users.validation.ts)       # User input validation
│   │   ├── [users.service.ts](mdc:src/modules/users/users.service.ts)          # User business logic
│   │   └── resolvers/                # User query resolvers
│   └── shared/                       # Shared module utilities
│       ├── pagination/               # Pagination utilities
│       ├── filtering/                # Filter utilities
│       └── connections/              # Relay connection helpers
│
├── graphql/                          # GraphQL infrastructure
│   ├── schema/                       # Schema building and assembly
│   │   ├── [builder.ts](mdc:src/graphql/schema/builder.ts)                # Pothos builder with plugins
│   │   ├── [index.ts](mdc:src/graphql/schema/index.ts)                  # Schema assembly & export
│   │   ├── [inputs.ts](mdc:src/graphql/schema/inputs.ts)                 # Shared input types
│   │   ├── [scalars.ts](mdc:src/graphql/schema/scalars.ts)                # Custom scalars (DateTime)
│   │   └── plugins/                  # Schema plugins
│   ├── context/                      # GraphQL context
│   │   ├── [context.types.ts](mdc:src/graphql/context/context.types.ts)          # Context type definitions
│   │   ├── [context.factory.ts](mdc:src/graphql/context/context.factory.ts)        # Context creation
│   │   ├── [context.auth.ts](mdc:src/graphql/context/context.auth.ts)           # Authentication helpers
│   │   └── [context.utils.ts](mdc:src/graphql/context/context.utils.ts)          # Context utilities
│   ├── directives/                   # Custom GraphQL directives
│   └── middleware/                   # GraphQL-specific middleware
│       ├── [shield-config.ts](mdc:src/graphql/middleware/shield-config.ts)          # Permission mapping
│       ├── [rules.ts](mdc:src/graphql/middleware/rules.ts)                  # Permission rules
│       └── [rule-utils.ts](mdc:src/graphql/middleware/rule-utils.ts)             # Rule helper functions
│
├── core/                             # Core business logic and utilities
│   ├── auth/                         # Authentication core
│   │   ├── [scopes.ts](mdc:src/core/auth/scopes.ts)                 # Authorization scopes
│   │   └── [types.ts](mdc:src/core/auth/types.ts)                  # Auth types
│   ├── errors/                       # Error handling system
│   │   ├── [types.ts](mdc:src/core/errors/types.ts)                  # Error class hierarchy
│   │   ├── [handlers.ts](mdc:src/core/errors/handlers.ts)               # Error normalization
│   │   └── [constants.ts](mdc:src/core/errors/constants.ts)              # Error messages
│   ├── logging/                      # Logging system
│   │   ├── [logger-factory.ts](mdc:src/core/logging/logger-factory.ts)         # Logger creation
│   │   └── [console-logger.ts](mdc:src/core/logging/console-logger.ts)         # Console implementation
│   ├── utils/                        # Core utilities
│   │   ├── relay.ts          # Global ID encoding/decoding
│   │   ├── [jwt.ts](mdc:src/core/utils/jwt.ts)                    # JWT utilities
│   │   ├── [crypto.ts](mdc:src/core/utils/crypto.ts)                 # Cryptographic utilities
│   │   ├── [dates.ts](mdc:src/core/utils/dates.ts)                  # Date utilities
│   │   ├── [strings.ts](mdc:src/core/utils/strings.ts)                # String utilities
│   │   └── [types.ts](mdc:src/core/utils/types.ts)                  # Type utilities
│   └── validation/                   # Validation system
│       ├── [schemas.ts](mdc:src/core/validation/schemas.ts)                # Common Zod schemas
│       └── [validators.ts](mdc:src/core/validation/validators.ts)             # Custom validators
│
├── data/                             # Data access layer
│   ├── database/                     # Database configuration
│   │   └── [client.ts](mdc:src/data/database/client.ts)                 # Prisma client setup
│   ├── loaders/                      # DataLoader implementations
│   │   └── [loaders.ts](mdc:src/data/loaders/loaders.ts)                # User/Post loaders
│   ├── repositories/                 # Repository pattern
│   │   └── [refresh-token.repository.ts](mdc:src/data/repositories/refresh-token.repository.ts) # Refresh token storage
│   └── cache/                        # Caching layer
│       ├── [memory-cache.ts](mdc:src/data/cache/memory-cache.ts)           # In-memory cache
│       └── [redis-cache.ts](mdc:src/data/cache/redis-cache.ts)            # Redis cache (optional)
│
├── gql/                              # GraphQL client operations (for testing)
│   ├── [queries.ts](mdc:src/gql/queries.ts)                    # Query definitions
│   ├── [mutations.ts](mdc:src/gql/mutations.ts)                  # Mutation definitions
│   └── [mutations-auth-tokens.ts](mdc:src/gql/mutations-auth-tokens.ts)      # Auth token mutations
│
├── types/                            # Global type definitions
├── constants/                        # Application constants
├── [main.ts](mdc:src/main.ts)                           # Build entry point
└── [prisma.ts](mdc:src/prisma.ts)                         # Prisma client export
```

## Key Architectural Patterns

This section is detailed in separate rule files:
- [Pothos Patterns](mdc:.cursor/rules/01-pothos-patterns.mdc)
- [Testing Patterns](mdc:.cursor/rules/02-testing-patterns.mdc)
- [Error Handling](mdc:.cursor/rules/03-error-handling.mdc)
- [Authentication](mdc:.cursor/rules/04-authentication.mdc)

## Environment Variables

```bash
# Required
DATABASE_URL="file:./dev.db"     # SQLite database path
JWT_SECRET="your-secret-key"      # JWT signing secret

# Optional
BCRYPT_ROUNDS=10                  # Password hashing rounds (default: 10)
NODE_ENV="development"            # Environment mode
PORT=4000                         # Server port
HOST="localhost"                  # Server host
```

## Common Development Patterns

### Adding a New Module

1. Create module directory structure:

   ```
   src/modules/feature/
   ├── feature.schema.ts       # GraphQL type definitions
   ├── feature.permissions.ts  # Authorization rules
   ├── feature.validation.ts   # Input validation
   ├── feature.service.ts      # Business logic (optional)
   ├── resolvers/
   │   └── feature.resolver.ts # GraphQL resolvers
   └── types/
       └── feature.types.ts    # TypeScript types
   ```

2. Import resolver in schema index:

   ```typescript
   // src/graphql/schema/index.ts
   import '../../modules/feature/resolvers/feature.resolver'
   ```

3. Add permissions to shield config:
   ```typescript
   // src/graphql/middleware/shield-config.ts
   import { featurePermissions } from '../../modules/feature/feature.permissions'
   ```

### DataLoader Usage

DataLoaders are created in context for N+1 prevention:

```typescript
// In context.utils.ts
loaders: {
  users: createUserLoader(),
  posts: createPostLoader(),
}
```

## Debugging Quick Reference

- **Type errors**: `bun run generate` (regenerates Prisma & GraphQL types)
- **Permission denied**: Check JWT token and shield-config.ts mappings
- **Global ID errors**: Verify Base64 encoding (e.g., "UG9zdDox" = "Post:1")
- **Database issues**: `bunx prisma studio` for visual inspection
- **GraphQL schema**: `bun run gen:schema` to update schema.graphql
- **Test specific operation**: Use GraphQL Playground at http://localhost:4000

## Recent Architecture Changes

1. **Modular Structure**: Migrated from scattered files to organized modules
2. **Direct Resolvers**: Removed abstraction layers, business logic in resolvers
3. **Improved Error Handling**: Centralized error types with consistent messages
4. **Enhanced Testing**: All tests use typed GraphQL operations from src/gql/
5. **TypeScript Strict**: All files pass strict type checking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/semyenov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
