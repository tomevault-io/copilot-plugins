## server-api-repository

> Generate a server API repository module


# Server API Repository Rules

## Naming Conventions
- File naming: Use `{entity-name}.repository.ts` in kebab-case
- Location: Place in `src/server/api/{entity-name}/` folder
- Function names: Use camelCase with verb + entity name pattern (e.g., `selectAllTools`, `insertUser`)
- SQL variable: Use camelCase with entity name + `Sql` suffix (e.g., `toolsSql`, `usersSql`)

## Required Structure
1. **SQL Commands Loading**:
   - Load SQL commands at the top using `readCommands("{entity-name}")`
   - Store the result in a constant named `{entityName}Sql`

2. **Core Functions**:
   - Export CRUD functions as needed for the entity
   - Common patterns:
     - `selectAll{EntityName}s` - Get all entities
     - `select{EntityName}ById` - Get entity by ID
     - `insert{EntityName}` - Create new entity
     - `update{EntityName}` - Update existing entity
     - `select{EntityName}By{Field}` - Get entity by specific field

3. **Return Types**:
   - Be explicit with return types
   - Return entity arrays as `EntityType[]`
   - Return single entities as `EntityType`
   - Return IDs as `number`

## Imports
- Import entity types from domain folder
- Import validation functions from entity type files
- Import SQL utilities from shared folder
- Import `AppError` if throwing custom errors

## Error Handling
- Validate input using entity validation function before insert/update
- Use `AppError` with appropriate error type for failures
- Handle special cases (e.g., duplicate records, not found)

## Documentation
- Add JSDoc comments to all exported functions
- Document parameters with types and descriptions
- Document return values
- Document exceptions that may be thrown

## Example Structure
```typescript
import type { EntityName } from "@/server/domain/entity-name.type";
import { validateEntityName } from "@/server/domain/entity-name.type";
import type { Raw } from "@/server/shared/sql.type";
import { AppError } from "@/server/shared/app-error.class";
import { 
  insert, 
  readCommands, 
  selectAll, 
  selectById 
} from "@/server/shared/sql.utils";

const entitySql = await readCommands("entity-name");

/**
 * Selects all entities
 * @returns The entities array
 */
export const selectAllEntities = (): EntityName[] => {
  const results = selectAll<EntityName>(entitySql.SELECT_ALL);
  return results || [];
};

/**
 * Selects an entity by id
 * @param id - The id of the entity
 * @returns The entity
 * @throws AppError if the entity is not found
 */
export const selectEntityById = (id: number): EntityName => {
  const result = selectById<EntityName>(entitySql.SELECT_BY_ID, id);
  return result;
};

/**
 * Inserts an entity
 * @param entityToInsert - The entity to insert
 * @returns The entity inserted
 * @throws AppError if the entity is not valid
 */
export const insertEntity = (entityToInsert: Raw<EntityName>): EntityName => {
  validateEntityName(entityToInsert);
  const entityId = insert<Raw<EntityName>>(entitySql.INSERT, entityToInsert);
  const entity = selectById<EntityName>(entitySql.SELECT_BY_ID, entityId);
  return entity;
};
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
