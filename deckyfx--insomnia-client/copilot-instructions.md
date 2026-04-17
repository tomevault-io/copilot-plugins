## insomnia-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About this Project

This is an HTTP request testing tool that is compliant with Insomnia configuration format. The program reads Insomnia exported YAML files and enables API testing with their descriptions. Key features include:

- **Insomnia Compatibility**: Reads and processes Insomnia exported YAML configurations
- **API Testing**: Execute HTTP requests with proper validation and response handling
- **Request Chaining**: Supports Insomnia's request chaining capabilities for complex workflows
- **Future MCP Integration**: Will be expanded into an MCP (Model Context Protocol) tool for API testing

The tool allows developers to leverage their existing Insomnia configurations in a CLI environment, maintaining compatibility with advanced features like environment variables, request dependencies, and automated testing workflows.

## Runtime and Package Management

This project uses Bun as the JavaScript runtime and package manager. Always use Bun commands instead of Node.js, npm, or other alternatives:

- **Install dependencies**: `bun install`
- **Run the application**: `bun run index.ts` or `bun index.ts`
- **Run with hot reload**: `bun --hot index.ts`
- **Run tests**: `bun test`
- **Build**: `bun build <file>`

## Project Structure

This is a TypeScript project configured with:
- Module system: ESM with `"type": "module"` in package.json
- TypeScript target: ESNext with bundler module resolution
- Strict TypeScript configuration enabled
- Entry point: `index.ts`

## Bun-Specific APIs and Patterns

When writing code, prefer Bun's built-in APIs over external packages:
- Use `Bun.serve()` for HTTP servers (supports WebSockets, HTTPS, routes)
- Use `bun:sqlite` for SQLite instead of `better-sqlite3`
- Use `Bun.redis` for Redis instead of `ioredis`
- Use `Bun.sql` for Postgres instead of `pg`
- Use built-in `WebSocket` instead of `ws` package
- Use `Bun.$\`command\`` for shell commands instead of `execa`
- Bun automatically loads `.env` files - no need for `dotenv`

## Frontend Development

If building frontend components, use HTML imports with `Bun.serve()`. HTML files can directly import `.tsx`, `.jsx`, or `.js` files, and Bun will handle transpilation and bundling automatically.

## Developer Guidelines

### Code Documentation
- **Always add inline comments** to explain complex logic, business rules, and non-obvious code sections
- **Add TSDoc comments** for every class and function with exactly one example usage
- Document parameters, return values, and any thrown errors

### Code Organization
- **Modularize code** as much as possible to avoid repetitive code
- Extract common functionality into utility functions or modules
- Use clear, descriptive function and variable names
- Keep functions small and focused on a single responsibility

### File Change Management
- **Always add a changelog comment** at the top of any file when editing it
- Format: `// CHANGELOG: [YYYY-MM-DD] - Brief description of changes made`
- This helps track modifications and understand the evolution of code

### TypeScript Quality
- **Ensure clean TypeScript compilation** - All code must pass `bun run typecheck` without errors
- Handle `strictNullChecks` properly - check for null/undefined values before using
- Use proper type guards and error handling for unknown types
- Fix all TypeScript warnings and errors before committing code

### Type Organization
- **All types and interfaces must go to `./src/@types/`** - Centralize type definitions for better organization and reusability
- **No `.js` extensions in TypeScript imports** - Use clean import paths without file extensions when importing TypeScript files
- **Separate types from implementations** - Keep interface/type definitions in `@types` directory while class implementations remain in their logical modules
- **Maintain export compatibility** - Use index files to re-export types and maintain API compatibility

### Communication and Planning
- **Explain before executing** - Always explain what you plan to do and why before implementing
- **Propose approach** - Describe your implementation strategy and any alternative approaches considered
- **Seek feedback** - Present your plan and ask for confirmation or suggestions before proceeding
- **Share reasoning** - Explain the rationale behind technical decisions and trade-offs
- **Suggest improvements** - Offer additional enhancements or optimizations when relevant

### Example Documentation Format
```typescript
/**
 * Processes user authentication data and returns a JWT token
 * @param userData - The user credentials object
 * @param options - Authentication options including expiry
 * @returns Promise resolving to JWT token string
 * @throws {AuthenticationError} When credentials are invalid
 * @example
 * ```typescript
 * const token = await authenticateUser(
 *   { email: "user@example.com", password: "secret" },
 *   { expiresIn: "24h" }
 * );
 * ```
 */
async function authenticateUser(userData: UserCredentials, options: AuthOptions): Promise<string> {
  // Validate input credentials before processing
  if (!userData.email || !userData.password) {
    throw new AuthenticationError("Missing required credentials");
  }
  // ... rest of implementation
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deckyfx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
