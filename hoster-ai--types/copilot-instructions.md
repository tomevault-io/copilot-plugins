## types

> The LLM documentation system is organized into a main index file (`llm.txt`) and detailed component files in the `llm/` folder. This structure keeps the main index lightweight while providing comprehensive details in separate files.


The LLM documentation system is organized into a main index file (`llm.txt`) and detailed component files in the `llm/` folder. This structure keeps the main index lightweight while providing comprehensive details in separate files.

## Structure

- **`llm.txt`**: Main index file containing titles, descriptions, source references, and links to detail files
- **`llm/dtos/`**: Individual markdown files for each DTO
- **`llm/enums/`**: Individual markdown files for each enum
- **`llm/decorators/`**: Individual markdown files for each custom decorator
- **`llm/validators/`**: Individual markdown files for each validator function
- **`llm/transformers/`**: Individual markdown files for each transformer function

## Main Index Format (`llm.txt`)

The main index should be organized by category (DTOS, ENUMS, DECORATORS, VALIDATORS, TRANSFORMERS) with each entry following this format:

```markdown
### ComponentName
**Description:** Brief description of the component

**Source:** `path/to/source.ts`

**Details:** See [llm/category/filename.md](llm/category/filename.md)

---
```

## Detail File Format (`llm/category/filename.md`)

Each component's detail file should follow this structure:

```markdown
# ComponentName

**Description:** Detailed description of the component

**Source:** `path/to/source.ts`

**Language:** typescript

## Code

```typescript
// Full code with comments, excluding JSONSchema decorators
```
```

## Content Guidelines

- **Keep content minimal and accurate**: Include DTO class shapes, enums, custom decorators, and validators that affect validation logic
- **Exclude framework-specific metadata**: Do NOT include `JSONSchema` imports or `@JSONSchema(...)` decorators in code blocks
- **Mirror sources**: Code blocks should reflect the actual source file structure, minus JSONSchema annotations
- **Allowed decorators**: `class-validator`, `class-transformer`, and project custom decorators (e.g., `@AtLeastOneNonEmptyProperty`, `@IsOneOf`, `@IsOfAllowedTypes`, etc.)
- **Consistency**: Ensure imports within code blocks include everything needed for the snippet to be accurate (e.g., `Type`, custom validators), but avoid runtime-only wiring

## Maintenance Workflow

When adding or updating a component:

1. Create or update the detail file in the appropriate `llm/category/` folder
2. Update the main `llm.txt` index with the component's entry
3. Ensure the detail file follows the format above
4. Remove all `@JSONSchema` decorators and imports from the code block
5. Keep all validation decorators and custom decorators

These guidelines apply only to the LLM documentation system. The TypeScript sources in `dtos/`, `validators/`, etc., may continue to use `class-validator-jsonschema` for schema generation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoster-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
