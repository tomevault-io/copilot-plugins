## server-domain-entity

> Rules to generate a server domain entity


# Server Domain Entity Rules

## Naming Conventions
- **File naming**: Use `{entity-name}.type.ts` in `kebab-case`
- **Type names**: Use `PascalCase` for type definitions (e.g., `Tool`)
- **Constants**: Use `UPPER_SNAKE_CASE` for null/default objects (e.g., `NULL_TOOL`)
- **Functions**: Use `camelCase` for validation functions (e.g., `validateTool`)
- **Fields**: Use `snake_case` for all entity properties (e.g., `user_id`)

## Required Exports
1. **Main Entity Type**:
   - Define as TypeScript `type` (not interface or class)
   - Include `id` as primary key
   - Include audit fields: `created_at` and `updated_at`

2. **NULL/Default Object**:
   - Export a constant with default values for all fields 
   - Name as `NULL_{ENTITY_NAME}`

3. **Validation Function**:
   - Export a function named `validate{EntityName}`
   - Accept a partial entity as parameter
   - Throw `AppError` with type "LOGIC" when validation fails

## Imports
- Import `AppError` from `@server/shared/app-error.class`
- Import related types from `@server/shared/sql.type`

## Extras
- Follow [general typesscript rules]()

## Example Structure
```typescript
import { AppError } from "../shared/app-error.class";

/**
 * Represents an entity with its properties
 */
export type EntityName = {
  id: number;
  name: string;
  // other properties...
  user_id: number;
  created_at: Date;
  updated_at: Date;
};

/**
 * Default empty entity object
 */
export const NULL_ENTITY_NAME: EntityName = {
  id: 0,
  name: "",
  // default values...
  user_id: 0,
  created_at: new Date(),
  updated_at: new Date(),
};

/**
 * Validates an entity
 * @param entity - The entity to validate
 * @throws AppError if the entity is invalid
 */
export const validateEntityName = (entity: Partial<EntityName>): void => {
  if (!entity.name /* other validations */) {
    throw new AppError("Invalid entity", "LOGIC");
  }
};
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
