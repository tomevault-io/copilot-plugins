## dockerutilities

> Go coding standards and best practices for this Docker utilities project


# Go Coding Standards

This project follows strict Go best practices and conventions:

## General Standards
- Use explicit error returns and handle all errors
- Follow idiomatic Go naming conventions (camelCase for variables, PascalCase for exported)
- Include comprehensive godoc comments for all exported types, functions, and constants
- Use `gofmt` for code formatting
- Export only what is intended for external use
- Use dependency injection for modularity

## Project Structure
- All implementation logic goes in `src/` directory organized into unique packages
- CLI commands are defined in `cmd/` directory using Cobra library
- Delegate business logic to packages in `src/`
- Keep packages focused and single-purpose

## Error Handling
- Always check and handle errors explicitly
- Return errors rather than panicking
- Use meaningful error messages
- Consider wrapping errors with context when appropriate

## Documentation
- Write package-level overview describing purpose and usage
- Include detailed comments for all exported items
- Provide example usage code where applicable
- Clear parameter and return value descriptions
- Place comments above declarations per Go conventions

## Testing
- Use Go's testing package (no external assert libraries)
- Cover all exported functions in the package
- Include table-driven tests where applicable
- Follow Go naming conventions (e.g., TestFunctionName)
- Mock dependencies using interfaces
- Test both success and error cases

## Imports
- Use explicit imports (critical for Go's explicit imports)
- Avoid dependencies outside stdlib or explicitly defined packages
- Group imports: stdlib, third-party, local
- Remove unused imports

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smiller333) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
