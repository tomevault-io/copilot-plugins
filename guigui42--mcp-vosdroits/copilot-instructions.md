## mcp-vosdroits

> Documentation standards for Go projects


# Documentation Standards

## Code Documentation

### General Principles

- Prioritize self-documenting code through clear naming
- Document all exported symbols (types, functions, methods, constants)
- Start documentation with the symbol name
- Write documentation in English
- Write complete sentences

### Package Documentation

- Add package comment at the top of one file in the package
- Start with "Package [name]"
- Explain the package's purpose and main functionality
- Include usage examples when helpful

Example:
```go
// Package tools provides MCP tool implementations for searching
// French public service information from service-public.gouv.fr.
package tools
```

### Function Documentation

- Document what the function does, not how it does it
- Include parameter descriptions when not obvious
- Document return values, especially errors
- Mention any side effects or special behavior

Example:
```go
// SearchProcedures searches for procedures on service-public.gouv.fr
// matching the given query. It returns up to limit results.
// Returns an error if the HTTP request fails or the response is invalid.
func SearchProcedures(ctx context.Context, query string, limit int) ([]Result, error)
```

### Type Documentation

- Document the purpose of the type
- Explain important fields if not obvious
- Document JSON/JSONSCHEMA tags meaning when complex

### Constant Documentation

- Document groups of related constants
- Explain the purpose and valid values

## README Documentation

### Required Sections

1. **Project Title and Description**
   - Clear, concise description of what the project does
   
2. **Installation**
   - How to install/build the project
   - Prerequisites and dependencies
   
3. **Usage**
   - Quick start guide
   - Basic examples
   - Common use cases
   
4. **Configuration**
   - Environment variables
   - Configuration file format
   - Default values
   
5. **API/Tool Documentation**
   - List of available MCP tools
   - Input/output schemas
   - Example requests/responses

6. **Development**
   - How to set up development environment
   - How to run tests
   - How to contribute

7. **Docker**
   - How to build the Docker image
   - How to run the container
   - Available tags

8. **License**
   - License information

### Code Examples

- Include working code examples
- Show both successful and error handling cases
- Use realistic data in examples
- Keep examples concise but complete

## Inline Comments

### When to Comment

- Explain complex algorithms or logic
- Clarify non-obvious business rules
- Document workarounds or temporary solutions
- Explain why, not what (code should be self-explanatory)

### When Not to Comment

- Don't comment obvious code
- Don't leave commented-out code
- Don't write comments that duplicate the code
- Avoid TODO comments in production code

## API Documentation

### JSON Schema Documentation

Use comprehensive `jsonschema` tags:
- `description` - Explain the field's purpose
- `required` - Mark required fields
- `example` - Provide example values when helpful
- Document constraints (min, max, format)

Example:
```go
type SearchInput struct {
    Query string `json:"query" jsonschema:"required,description=Search query for procedures,example=carte d'identité"`
    Limit int    `json:"limit,omitempty" jsonschema:"minimum=1,maximum=100,description=Maximum number of results,default=10"`
}
```

## Changelog

- Maintain a CHANGELOG.md file
- Follow [Keep a Changelog](https://keepachangelog.com/) format
- Document all notable changes
- Group changes by version
- Use semantic versioning

## Error Messages

- Write clear, actionable error messages
- Include context about what failed
- Suggest how to fix the problem when possible
- Use proper grammar and punctuation

---
> Source: [guigui42/mcp-vosdroits](https://github.com/guigui42/mcp-vosdroits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
