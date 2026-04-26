## ai-cli-chat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

It includes information about the architecture, commands, code style, and testing standards to ensure consistency and maintainability.

It is important to follow these guidelines to maintain a high standard of code quality and ensure that the application is easy to understand and extend.

## Architecture

- TypeScript and Node.js application
- Modular design with clear separation of concerns
- Uses Vercel AI SDK for AI model interaction
  - [Vercal AI Core](https://sdk.vercel.ai/docs/ai-sdk-core)
- Uses @clack/core and @clack/prompts for command-line interface:
  - [@clack/core](https://github.com/bombshell-dev/clack/tree/main/packages/core#readme)
  - [@clack/prompts](https://github.com/bombshell-dev/clack/tree/main/packages/prompts#readme)
- Follows best practices for command-line applications
- Implements a streaming chat interface in the terminal
- Supports multiple AI models
- Includes chat history in each request
- Differentiates AI output and user input using the style function from @clack/prompts
- Uses environment variables for configuration
- Uses dotenv for loading environment variables
- Uses Vitest for testing
- Uses ESLint and Prettier for code style

## Commands

- Build: `npm run build`
- Dev: `npm run dev`
- Start: `npm start`
- Lint: `npm run lint`
- Test all: `npm test`
- Test single file: `npm test tests/filename.test.ts`
- Test watch mode: `npm run test:watch`
- Test coverage: `npm run test:coverage`

## Code Style

- TypeScript with strict typing; avoid `any`
- Use ESLint and Prettier for code style
  - Run `npm run format` before committing
- Use async/await for asynchronous code
- Import order: built-in modules, external libs, internal modules
- Name components and functions with PascalCase and camelCase
- For CLI components, follow @clack/prompts conventions
- Error handling: use try/catch with meaningful error messages
- Prefer functional programming patterns
- Document public functions with JSDoc comments
- Log chat history to a file while respecting privacy
- Use environment variables for API keys and configuration
- Use `dotenv` for loading environment variables
- Use `tsconfig.json` for TypeScript configuration

## Testing Standards

- All tests must be written in TypeScript
- Every module must have corresponding test files
- Achieve high test coverage for all core functionality
- Mock external dependencies (fs, APIs, etc.)
- Test both success and error paths
- Use descriptive test names following "it should..." pattern
- Organize tests using describe blocks for clear grouping
- Prefer small, focused unit tests over large integration tests
- Use beforeEach to set up and reset mocks between tests
- Follow AAA pattern: Arrange, Act, Assert
- Use Jest as the testing framework

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nnance) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
