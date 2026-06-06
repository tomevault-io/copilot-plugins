## server-api-controller

> Generate a server API controller module


# Server API Controller Rules

## Naming Conventions
- File naming: Use `{entity-name}.controller.ts` in kebab-case
- Location: Place in `src/server/api/{entity-name}/` folder
- Routes object: Name as `{entityName}Routes` (camelCase)
- Controller functions: Use camelCase with HTTP method + entity name pattern (e.g., `getTools`, `postAsset`)
- Response types: Use PascalCase with entity name + "Response" suffix (e.g., `AssetResponse`, `UserTokenResponse`)
- Request types: Use PascalCase with entity name + "Request" suffix (e.g., `CredentialsRequest`, `ToolPostRequest`)

## Required Structure
1. **Routes Object**:
   - Export a constant named `{entityName}Routes`
   - Map HTTP methods (GET, POST, PUT, DELETE) to their handler functions
   - Include JSDoc comment describing available routes

2. **Controller Functions**:
   - Implement private controller functions for each HTTP method
   - Follow naming pattern: `{httpMethod}{EntityName}` (e.g., `getTools`, `postAsset`)
   - Return Response objects using shared response utilities

3. **Request Processing**:
   - Use guard functions for validation (`guardGetUserId`, `guardGetBody`)
   - Extract and validate request bodies
   - Transform DTOs to domain objects before calling repositories

4. **Response Formatting**:
   - Use response utilities (e.g., `ok`, `badRequest`) 
   - Map domain entities to response DTOs when needed
   - Maintain consistent response structure

## DTO Generation
1. **Request DTOs**:
   - File naming: Use `{entity-name}-{action}-request.type.ts` in kebab-case (e.g., `asset-post-request.type.ts`)
   - Type naming: Use PascalCase with action + "Request" suffix (e.g., `AssetPostRequest`, `CredentialsRequest`)
   - Include only fields needed from the client, omitting server-specific fields like `user_id`, `created_at`, etc.
   - Example:
     ```typescript
     /**
      * Request DTO for creating a new entity
      */
     export type EntityPostRequest = {
       name: string;
       description: string;
       category_id: number;
       // Omit user_id, created_at, updated_at, etc.
     };
     ```

2. **Response DTOs**:
   - File naming: Use `{entity-name}-response.type.ts` in kebab-case (e.g., `asset-response.type.ts`, `user-token-response.type.ts`)
   - Type naming: Use PascalCase with "Response" suffix (e.g., `AssetResponse`, `CategoryResponse`)
   - Typically mirror the domain entity, but may exclude sensitive or unnecessary fields
   - May include additional derived or computed properties not in the domain model
   - Example:
     ```typescript
     /**
      * Response DTO for entity data
      */
     export type EntityResponse = {
       id: number;
       name: string;
       description: string;
       category_id: number;
       created_at: string; // Often date fields are stringified in responses
       // May omit internal fields like user_id
     };
     ```

3. **DTO Transformations**:
   - Use type casting with `as` for simple transformations
   - For complex transformations, create mapping functions
   - Handle date serialization (converting Date objects to strings)
   - Example:
     ```typescript
     // Simple mapping
     const response: EntityResponse = entity as EntityResponse;
     
     // Complex mapping with multiple entities
     const response: EntityResponse[] = entities.map(entity => ({
       id: entity.id,
       name: entity.name,
       description: entity.description,
       category_id: entity.category_id,
       created_at: entity.created_at.toISOString()
     }));
     ```

## Imports
- Import entity types from domain folder
- Import repository functions from the corresponding repository file
- Import request/response utilities from shared folder
- Import request/response type definitions

## Authentication
- Use `guardGetUserId` for secured endpoints
- Apply user ID to created entities via the `user_id` field
- Handle authorization checks as needed

## Error Handling
- Use appropriate response utilities for error cases
- Consider validation before processing

## Documentation
- Add JSDoc comments to the routes object
- Document the available HTTP methods and their purpose
- Add JSDoc comments to all DTO types

## Example Structure
```typescript
import type { Entity } from "@/server/domain/entity.type";
import { guardGetBody, guardGetUserId } from "@/server/shared/request.utils";
import { ok } from "@/server/shared/response.utils";
import type { Raw } from "@/server/shared/sql.type";
import type { EntityPostRequest } from "./entity-post-request.type";
import type { EntityResponse } from "./entity-response.type";
import { insertEntity, selectAllEntities } from "./entity.repository";

/**
 * Routes controller for /api/entities
 * - GET: Get all entities
 * - POST: Create a new entity
 * @description Object that wires the request to the correct controller
 */
export const entityRoutes = {
  GET: async (request: Request): Promise<Response> => await getEntities(request),
  POST: async (request: Request): Promise<Response> => await postEntity(request),
};

const getEntities = async (request: Request): Promise<Response> => {
  const entities = selectAllEntities();
  const response: EntityResponse[] = entities.map(e => e as EntityResponse);
  return ok<EntityResponse[]>(response);
};

const postEntity = async (request: Request): Promise<Response> => {
  const userId = guardGetUserId(request);
  const entityDto = await guardGetBody<EntityPostRequest>(request);
  
  const rawEntity: Raw<Entity> = {
    ...entityDto,
    user_id: userId,
  };
  
  const entity = insertEntity(rawEntity);
  const response: EntityResponse = entity as EntityResponse;
  
  return ok<EntityResponse>(response);
};
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
