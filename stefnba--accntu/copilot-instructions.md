## accntu

> Guidelines for backend development

# Backend Development Rules

This project uses Hono as the main backend framework with a carefully designed architecture for error handling, database operations, and API endpoints.

## Tech Stack Overview

- **API Framework**: [Hono](mdc:https:/hono.dev/docs)
- **Database ORM**: Drizzle
- **Validation**: Zod
- **Error Handling**: Custom error handling system

## Backend Architecture

The server codebase follows a feature-based organization with clear separation of concerns:

```
src/server/
├── db/                # Database configuration and schemas
├── features/          # Feature-based organization
├── lib/               # Shared libraries
├── middleware/        # Shared middleware
└── index.ts           # Main Hono API setup
```

## Core Development Guidelines

### API Routes & Middleware

- Use Hono for all API routes, not Next.js server actions
- Implement proper middleware for auth, error handling, etc.
- Follow the established route handler patterns

@file:src/server/index.ts
@file:src/server/README.md

### Error Handling

The project has a comprehensive error handling system. Follow these guidelines:

- Always use the error factory for creating errors
- Use appropriate error types based on where the error occurs
- Include proper error codes and status codes
- Chain errors when appropriate for better debugging

@file:src/server/lib/error/README.md
@file:src/server/lib/error/ROUTE_HANDLERS.md

### Database Operations

- Use Drizzle as the ORM
- Always use the handler utilities for database operations
- Provide operation descriptions
- Use proper error handling for database operations

@file:src/server/db/README.md
@file:src/server/lib/handler/README.md

### Validation

- Define Zod schemas for all data validation
- Use the zValidator middleware for validating request data
- Handle validation errors properly

```typescript
import { zValidator } from '@/server/lib/validation';
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
});

app.post('/users', zValidator('json', UserSchema), async (c) => {
  const data = c.req.valid('json');
  // Process validated data
});
```

### API Responses

- Use createSuccessResponse for success responses
- Follow the established response format
- Use appropriate status codes

```typescript
import { createSuccessResponse } from '../lib/error/response';

return c.json(createSuccessResponse({ user }));
```

### Performance Optimization

Consider the performance optimization suggestions for larger applications:

- Implement query caching for frequently accessed data
- Use batch database operations when appropriate
- Add response compression for larger payloads
- Use proper caching headers

@file:src/server/OPTIMIZATION_SUGGESTIONS.md

## Feature Development

When developing new features, follow the comprehensive feature architecture guidelines:

@file:.cursor/rules/features.mdc

Key principles:
1. Use Hono for all API endpoints (never Next.js server actions)
2. Follow the established feature directory structure
3. Implement proper error handling with withRoute wrapper
4. Use Zod validation for all inputs
5. Follow the database schema and query patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stefnba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stefnba)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
