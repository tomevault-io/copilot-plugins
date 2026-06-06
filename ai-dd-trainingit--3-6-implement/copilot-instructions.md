## client-repository

> Generate a client repository


# Client Repository Rules

## Naming Conventions
- File naming: Use `{entity-name}.repository.ts` in kebab-case
- Location: Place in `src/client/repositories/` folder
- Function names: Use camelCase with verb + entity name pattern (e.g., `getAllTools`, `createUser`)
- API base URL: Define as a private constant named `BASE_URL`

## Required Structure
1. **Core Functions**:
   - Export CRUD functions as needed for the entity
   - Common patterns with REST verbs:
     - `getAll{EntityName}s` - Get all entities
     - `get{EntityName}ById` - Get entity by ID
     - `post{EntityName}` - Create new entity
     - `put{EntityName}` - Update existing entity
     - `delete{EntityName}` - Delete entity
     - `get{EntityName}By{Field}` - Get entity by specific field

2. **Return Types**:
   - Be explicit with return types
   - Return entity arrays as `EntityType[]`
   - Return single entities as `EntityType`
   - Use Promise for async operations

3. **Request Methods**:
   - Use the fetch utilities in `/src/client/shared/fetch.utils.ts`
   - Handle HTTP methods (GET, POST, PUT, DELETE)
   - Transform API responses to client entity objects

## Cache Management (Optional)
- Implement simple caching mechanism if needed
- Clear cache on mutations (create, update, delete)
- Use localStorage for persistence if appropriate

## Imports
- Import entity types from domain folder
- Import fetch utilities
- Import validation functions if needed

## Error Handling
- Handle network errors and API errors
- Implement retry logic if necessary
- Transform API errors to user-friendly messages

## Documentation
- Add JSDoc comments to all exported functions
- Document parameters with types and descriptions
- Document return values and promise resolution
- Document error cases

## Example Structure
```typescript
import { get, post } from "../shared/fetch.utils";
import type { EntityName } from "../domain/entity-name.type";

const BASE_URL = "/api/entities";

export const getAllEntities = async (): Promise<EntityName[]> => {
  const response = await  get<EntityName[]>(BASE_URL);
  return response || [];
};

export const getEntityById = async (id: number): Promise<EntityName> => {
  const response = await get<EntityName>(`${BASE_URL}/${id}`);
  return response;
};

export const createEntity = async (entity: unknown): Promise<EntityName> => {
  const response = await post<EntityName>(BASE_URL, entity);
  return response;
};

```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
