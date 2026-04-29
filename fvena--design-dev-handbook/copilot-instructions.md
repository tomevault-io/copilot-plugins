## design-dev-handbook

> Guidelines for Module Design and Public API


# Module Design and Public API

These rules provide guidelines for module design, code organization, and public API design for this TypeScript library.

## Module Structure

- Organize code into modules with single, cohesive purpose
- Use barrel files (index.ts) for public exports
- Maintain clear separation between public API and implementation details
- Structure directories by domain or feature, not by type
- Implement clear boundaries between modules with well-defined interfaces
- Maintain unidirectional dependencies between modules
- Avoid circular dependencies at all costs
- Use subpath exports to expose specific modules

## Public API Design

- Design intuitive APIs that are difficult to use incorrectly
- Follow the principle of least surprise in interface design
- Implement function overloading to improve usability
- Provide sensible defaults for optional parameters
- Maintain backward compatibility in updates
- Version breaking changes in the public API correctly
- Document the public API completely with JSDoc
- Include usage examples for all public functions

## Export Conventions

- Export only what is part of the public API
- Use named exports for most functionality
- Implement default exports only when it makes sense
- Avoid re-exports that hide the module origin
- Clearly distinguish between public and internal exports
- Use type exports for types only used at compile time
- Maintain consistency in export conventions
- Avoid wildcard exports (export *)

## Interface Design

- Create cohesive interfaces with single purpose
- Implement small, composable interfaces
- Follow interface segregation principle (ISP)
- Design for extensibility when appropriate
- Use utility types for variations (Partial, Pick, etc.)
- Avoid overly generic or ambiguous interfaces
- Document the purpose and usage of each interface
- Implement consistent interfaces across related modules

## Module Patterns

- Use the revealing module pattern when appropriate
- Implement lazy initialization for heavy modules
- Use factory functions to create module instances
- Implement singleton for global services when necessary
- Export types alongside their implementations
- Create namespaces to group related functionality
- Use import type for imports used only in types
- Implement dynamic imports for code splitting when beneficial

## Configuration Design

- Create configuration options with sensible defaults
- Implement validation for configuration options
- Use partial application to configure components
- Provide builders or factories for complex configuration
- Implement configuration merging with defaults
- Document all configuration options
- Preserve immutability in configuration options
- Implement type validation for configurations

## Code Organization

- Structure code by features or domains
- Place tests alongside implementations when appropriate
- Group related types with their implementations
- Keep files focused on a single responsibility
- Implement clear boundaries between application layers
- Maintain consistency in structure across similar modules
- Use directories to group related features
- Implement path aliases for cleaner imports

## Versioning and Stability

- Follow semantic versioning strictly
- Clearly mark experimental or unstable APIs
- Implement deprecation warnings before removing functionality
- Document breaking changes thoroughly
- Maintain backward compatibility when possible
- Provide migration tools for major changes
- Use types to ensure compatibility in updates
- Implement version coexistence when necessary

## Extensibility

- Design for extension, closed for modification (Open/Closed)
- Implement hooks or extension points at strategic locations
- Use interfaces to allow alternative implementations
- Provide customizable factory functions
- Implement plugin or middleware system when beneficial
- Design for composition over excessive configuration
- Document extension points and recommended patterns
- Provide examples of common extensions

## Interoperability

- Design for interoperability with TypeScript/JavaScript ecosystem
- Implement compatibility with common ecosystem patterns
- Provide adapters for integration with other libraries
- Use standard types when available
- Implement interoperability with web APIs when relevant
- Design for both Node.js and browser environments when possible
- Provide types for JavaScript consumers
- Maintain compatibility with different module systems

## Module Documentation

- Document purpose and responsibilities of each module
- Include usage examples for main modules
- Document relationships between modules
- Provide architecture diagrams when useful
- Document recommended import patterns
- Include migration guides for significant changes
- Maintain detailed changelog by module
- Implement inline documentation with JSDoc

## Dos

- Design for API clarity and usability
- Structure modules with clear boundaries and responsibilities
- Keep public API minimal and focused
- Document public interfaces completely
- Favor composition over complex configuration
- Implement semantic versioning rigorously
- Design for extensibility where it adds value
- Maintain consistency in conventions across modules

## Don'ts

- Don't expose implementation details in public API
- Avoid monolithic modules with multiple responsibilities
- Don't change public API without appropriate version increment
- Avoid circular dependencies between modules
- Don't create excessively large or generic interfaces
- Avoid overly complex configurations
- Don't reuse names across different contexts or modules
- Avoid exporting utilities unrelated to module purpose

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fvena) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
