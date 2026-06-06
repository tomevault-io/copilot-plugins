## client-entity

> Generate a client domain entity


# Client Entity Rules

## Naming Conventions
- File naming: Use `{entity-name}.type.ts` in kebab-case
- Type names: Use PascalCase for type definitions (e.g., `Tool`)
- Constants: Use UPPER_SNAKE_CASE for null/default objects (e.g., `NULL_TOOL`)
- Functions: Use camelCase for validation functions (e.g., `validateTool`)
- Fields: Use snake_case for all entity properties (They are the couterparts of server DTOs)

## Required Exports
1. **Main Entity Type**:
   - Define as TypeScript `type` (not interface or class)
   - Include `id` as primary key
   - Include audit fields: `create_at` and `updated_at` (snake_case to match client conventions)

2. **NULL/Default Object**:
   - Export a constant with default values for all fields 
   - Name as `NULL_{ENTITY_NAME}`

3. **Validation Function** (optional):
   - Export a function named `validate{EntityName}`
   - Accept a partial entity as parameter
   - Return boolean or throw custom error

## Imports
- Import related entity types if needed

## Documentation
- Add JSDoc comments to all exported types, constants, and functions

## Example Structure
```typescript
/**
 * Represents a client entity with its properties
 */
export type EntityName = {
  id: number;
  name: string;
  // other properties...
  user_id: number; // Note snake_case convention for FK
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
  userId: 0,
  created_at: new Date(),
  updated_at: new Date(),
};

/**
 * Validates an entity
 * @param entity - The entity to validate
 * @returns true if valid, false otherwise
 */
export const validateEntityName = (entity: Partial<EntityName>): boolean => {
  // Client-side validation logic
  return Boolean(entity.name /* other validations */);
};
```

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
