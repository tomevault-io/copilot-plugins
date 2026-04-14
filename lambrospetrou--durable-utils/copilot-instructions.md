## durable-utils

> This package provides utilities and abstractions for Cloudflare [Durable Objects](https://developers.cloudflare.com/durable-objects/) and Cloudflare [Workers](https://developers.cloudflare.com/workers/).

# CLAUDE.md

This package provides utilities and abstractions for Cloudflare [Durable Objects](https://developers.cloudflare.com/durable-objects/) and Cloudflare [Workers](https://developers.cloudflare.com/workers/).

## Build & Test Commands
- Format code: `npm run fmt`
- Generate types: `npm run gen:types`
- Build project: `npm run build`
- Run all tests: `npm test`
- Run a single test: `npx vitest run <path-to-test-file> -t "<test-description>"`
- Release package: `npm run release`

## Code Style Guidelines
- Use TypeScript with strict mode enabled
- Use ESM module format (type: "module" in package.json)
- Format code with Prettier
- Write comprehensive JSDoc comments for public APIs
- Use camelCase for variables/functions, PascalCase for classes/types
- Error handling: Use type assertions for unknown types (as seen in isErrorRetryable)
- Each exported utility should have its own file
- Follow existing patterns when adding new features
- Tests should mirror the structure of source files
- Use Vitest with describe/it/expect pattern
- Include detailed type definitions for all public APIs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lambrospetrou)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lambrospetrou)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
