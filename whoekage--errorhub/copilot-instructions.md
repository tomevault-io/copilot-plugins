## errorhub

> In this project, we've established a pattern where we rely exclusively on Zod for schema validation rather than Fastify's built-in schema validation capabilities. This means:

# Fastify Validation Pattern

## Zod Only Approach

In this project, we've established a pattern where we rely exclusively on Zod for schema validation rather than Fastify's built-in schema validation capabilities. This means:

1. Route handlers should NOT include `schema` objects in route definitions
2. All data validation is handled through Zod schemas imported from the DTO layer
3. Validation happens explicitly in handlers rather than through Fastify's schema validation

## Implementation Pattern

Routes should be defined without schema objects:

```typescript
fastify.post<{
  Body: z.infer<typeof createErrorCodeRequest>;
  Reply: z.infer<typeof createErrorCodeResponse>;
}>(
  '/',
  async (request, reply) => {
    // Validate request body manually
    const validatedData = createErrorCodeRequest.parse(request.body);
    
    // Process the validated data using appropriate repository
    const result = await repositories.errorCode.create(validatedData);
    
    // Return response
    return reply.code(201).send(result);
  }
);
```

## Validation Steps

1. Import the relevant Zod schemas from the DTO layer
2. Create any additional schemas needed for route parameters or query strings
3. Define TypeScript types based on the schemas using `z.infer<typeof schema>`
4. Validate input data using `schema.parse()` or `schema.safeParse()` methods
5. Handle validation errors appropriately

## Error Handling

There are two approaches to error handling:

### Using parse()

```typescript
try {
  const validatedData = schema.parse(inputData);
  // Process validated data
} catch (error) {
  if (error instanceof z.ZodError) {
    return reply.code(400).send({
      statusCode: 400,
      error: 'Validation Error',
      message: error.errors[0].message,
      errors: error.errors
    });
  }
  throw error;
}
```

### Using safeParse()

```typescript
const result = schema.safeParse(inputData);
if (!result.success) {
  return reply.code(400).send({
    statusCode: 400,
    error: 'Validation Error',
    message: result.error.errors[0].message,
    errors: result.error.errors
  });
}
// Process result.data
```

## Current DI Container Structure

As of now, the DI container has the following structure:

```typescript
interface DIContainer {
  // Database
  db: DataSource;
  
  // Repositories
  repositories: {
    errorCode: IErrorCodeRepository;
    // Note: errorCategory and errorTranslation are not implemented yet
  };
  
  // Services
  services: {
    // Note: Services are not implemented yet
  };
}
```

When updating route files, please check which repositories/services are available in the DI container.

## DTO Organization

DTOs are organized in the `packages/api/src/dto` directory:

- [packages/api/src/dto/errors](mdc:packages/api/src/dto/errors) - Error-related DTOs
- [packages/api/src/dto/categories](mdc:packages/api/src/dto/categories) - Category-related DTOs
- [packages/api/src/dto/translations](mdc:packages/api/src/dto/translations) - Translation-related DTOs

Each domain has its own directory with consistent file naming:
- `base.dto.ts` - Common schema fields
- `create.dto.ts` - Schemas for creation operations
- `update.dto.ts` - Schemas for update operations
- `list.dto.ts` - Schemas for list operations

## Advantages

1. **Simplicity** - Only one validation system to learn and use
2. **Type Safety** - Zod provides excellent TypeScript integration
3. **Flexibility** - More control over validation flow and error handling
4. **Consistency** - Same validation approach used throughout the application

## Example Route Handler

Example from [packages/api/src/routes/errors/create.ts](mdc:packages/api/src/routes/errors/create.ts):

```typescript
import { FastifyInstance } from 'fastify';
import { DIContainer } from '@/di';
import { 
  createErrorCodeRequest,
  createErrorCodeResponse
} from '@/dto/errors';
import { z } from 'zod';

export default function(fastify: FastifyInstance, { repositories }: DIContainer) {
  fastify.post<{
    Body: z.infer<typeof createErrorCodeRequest>;
    Reply: z.infer<typeof createErrorCodeResponse>;
  }>(
    '/',
    async (request, reply) => {
      try {
        // Validate request body with Zod
        const validatedData = createErrorCodeRequest.parse(request.body);
        
        // Create the error code
        const newErrorCode = await repositories.errorCode.create(validatedData);
        return reply.code(201).send(newErrorCode);
      } catch (error) {
        if (error instanceof z.ZodError) {
          return reply.code(400).send({
            statusCode: 400,
            error: 'Validation Error',
            message: error.errors[0].message,
            errors: error.errors
          });
        }
        throw error; // Let Fastify handle unexpected errors
      }
    }
  );
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whoekage)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/whoekage)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
