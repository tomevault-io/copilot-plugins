## email

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build/Run/Test Commands
- Install dependencies: `npm install`
- Start the application: `npm start`
- Run tests: `npm test`
- Run a single test: `npm test -- -t "test name"` 
- Lint code: `npm run lint`

## Code Style Guidelines
- **Formatting**: Use 2-space indentation, no trailing whitespace, and semicolons
- **Imports**: Group imports by type (node modules first, then local modules)
- **Naming**: Use camelCase for variables/functions, PascalCase for classes/components
- **Types**: Use TypeScript with explicit types for function parameters and return values
- **Error Handling**: Use try/catch blocks for async operations, provide informative error messages
- **Documentation**: Add JSDoc comments for functions and components
- **Code Organization**: Split code into modular components with clear responsibilities

## Project Structure
This is a Node.js application following SOLID principles with the following structure:
- `src/application/useCases`: Business logic and use cases
- `src/domain/interfaces`: Contracts and interfaces
- `src/infrastructure/imap`: IMAP implementation using imapflow
- `src/presentation/cli`: Terminal UI using enquirer
- `src/shared`: Configuration and dependency injection

## Architecture Guidelines
- Follow SOLID principles and dependency injection pattern
- Use tsyringe for dependency injection
- Keep email provider implementation details in infrastructure layer
- Write tests with Jest using dependency mocks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/franciscoclavero)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/franciscoclavero)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
