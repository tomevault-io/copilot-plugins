## guidant

> This document outlines the core principles and guidelines for development in this workspace. These rules are intended to ensure consistency, quality, and maintainability of the codebase.


# Workspace Rules

This document outlines the core principles and guidelines for development in this workspace. These rules are intended to ensure consistency, quality, and maintainability of the codebase.

## Core Principles

### Primary Engineering Principles
- **Systematic Over Speed**: Always enforce proper research → plan → implement → test cycles.
- **Context Preservation**: Maintain complete project history and decision tracking.
- **Business-Focused UX**: Hide technical complexity, present only business choices to users.
- **Quality Gates**: Block progression until requirements are met.

### SOLID Principles
- **Single Responsibility Principle (SRP)**: Each module or class should have one, and only one, reason to change.
- **Open/Closed Principle (OCP)**: Software entities should be open for extension, but closed for modification.
- **Liskov Substitution Principle (LSP)**: Objects of a superclass shall be replaceable with objects of its subclasses without breaking the application.
- **Interface Segregation Principle (ISP)**: No client should be forced to depend on methods it does not use.
- **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

## Code Style & Architecture

### General
- **ES Modules**: Use `import/export` syntax.
- **Modern JavaScript**: Leverage `async/await`, destructuring, and template literals.
- **Type Safety**: Use JSDoc for type hints.

### Naming Conventions
- **Variables/Functions**: `camelCase`
- **Classes/Interfaces**: `PascalCase`
- **Constants**: `UPPER_CASE`
- **Files**: `kebab-case` for modules, `PascalCase` for classes.

### Import Organization
1.  External packages
2.  Internal modules by layer (domain, infrastructure, application)

## Documentation
- **JSDoc**: Document all public APIs.
- **Inline Comments**: Explain complex logic and architectural decisions.
- **READMEs**: Maintain up-to-date documentation for each major module.

## Error Handling
- Use `try/catch` blocks with specific error types.
- Return consistent result objects (e.g., `{ success, error, message, data }`).

## Testing
- **Test Runner**: Use `bun test`.
- **Location**: Test files should be in a `tests/` directory that mirrors the `src/` structure.
- **Structure**: Use `describe`, `it`, and `expect` from `bun:test`.
- **Coverage**: Aim for a minimum of 80% test coverage for business logic.

## Development Workflow
- **Research-First**: Use research tools before making architectural decisions.
- **Record Rationale**: Document the "why" behind major technical decisions.
- **Verify Completion**: Use the pre-completion checklist to ensure all criteria are met before marking a task as complete.
- **Provide Evidence**: Always provide concrete evidence of completion (e.g., test results, demos).

## Communication
- **Ask for Input**: Get user approval for major architectural changes and clarify ambiguous requirements.
- **Communicate Progress**: Provide regular status updates with evidence.
- **No Jargon**: Focus on business impact and user-facing benefits.
- **Surface Problems**: Report issues immediately.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/louisklinogo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
