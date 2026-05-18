## package-management

> Package Management Standards

- Use Bun as the primary package manager
- All scripts in package.json should use bun commands
- Use `bun install` for dependency installation
- Use `bun run` for running scripts
- Maintain a single bun.lockb file at the root
- Use workspace features for monorepo management if applicable
- Pin dependency versions for production dependencies
- Organize package.json sections consistently (dependencies, devDependencies, scripts, etc.)
- Include appropriate type declarations (@types/*) for all dependencies
- Regularly audit dependencies for security vulnerabilities with `bun pm audit`
- Document script usage in README.md or CONTRIBUTING.md
- Group related dependencies in package.json with comments
- Use Bun's built-in test runner for unit tests
- Leverage Bun's TypeScript support without the need for ts-node
- Optimize package.json scripts with Bun's fast execution capabilities

---
> Source: [sushaantu/boxento](https://github.com/sushaantu/boxento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
